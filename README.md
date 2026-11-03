# Antibody–Antigen Interaction Analyzer (full stack)

```
.
├── src/, public/, index.html, ...   React + Vite + TypeScript frontend (from Google AI Studio)
└── backend/                         Python FastAPI backend (real structural analysis)
```

## What the backend does
Given a PDB ID, it downloads the structure from RCSB (PDB, or mmCIF for large entries) and computes:
- antibody heavy/light chain detection and **IMGT-style CDR H1–H3 / L1–L3 mapping**
- **5 Å antibody–antigen contacts** with interaction typing (salt bridge, H-bond, π-stacking, electrostatic, vdW)
- the **20×20 residue interaction matrix** (rows = antigen aa, columns = antibody aa, order `ARNDCQEGHILKMFPSTWYV`, values 0–1)
- top residue pairs, interface statistics, matrix SHA-256 and a cognate score
and returns JSON in exactly the `PdbComplexData` shape from `src/types/biology.ts`.

## Run (two terminals)
**1. Backend** (Python 3.9+)
```bash
cd backend
python -m venv venv && source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000                   # docs: http://localhost:8000/docs
```
**2. Frontend** (Node.js 18+)
```bash
npm install
npm run dev                                             # http://localhost:3000
```
Vite forwards `/api/*` to `http://localhost:8000` (see `vite.config.ts`).
If the backend is off, the app falls back to the bundled benchmark entries and otherwise shows a clear error.
To point at a deployed backend, set `VITE_API_BASE_URL=https://your-backend` in `.env.local`.

## API
| Method | Path | Notes |
|---|---|---|
| GET | `/api/health` | health check |
| GET | `/api/analyze/{pdb_id}` | `?cutoff=5.0&heavy_chains=H&light_chains=L&antigen_chains=A` (overrides optional) |
| POST | `/api/analyze` | multipart upload of a `.pdb`/`.cif` file (same options as form fields) |

## Honest limitations
- **Cognate probability is a heuristic proxy**, not the trained CNN described on the site (the model isn't in this project). The UI labels it as such. To use the real model, edit `backend/scoring.py`.
- **ΔG values are rough per-interaction-type estimates**, not free-energy calculations.
- CDR boundaries are anchor-based approximations of IMGT numbering (exact for typical antibodies; use ANARCI for unusual germlines).
- Bundled benchmark entries (`src/data/benchmarkStructures.ts`) contain hand-written/illustrative numbers; they are used only when the backend is unreachable.
- The 3D viewer still fetches coordinates directly from RCSB, so uploaded (non-RCSB) files are analysed by the API but not drawn.

## Tests
```bash
cd backend && pip install pytest httpx && pytest tests
```
