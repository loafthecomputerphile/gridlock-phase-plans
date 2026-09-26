# Phase 01 — FOUNDATION (T+0 → T+2h)

**Mission:** scaffold the repo, environments, and data copy so every later phase starts from a working, committed baseline.

**Inputs:** `phase-plans\00-PROJECT-STATE.md` (read FIRST, always) · `research\Sperry-Tech-Challenge\` (source of the data copy) · report §Build Breakdown "Recommended stack" (`research\gridlock-evaluation.md`).

**Outputs:** `hackathon\gridlock\` repo on `main` with: Python env (uv), frontend scaffold (npm/Vite), `data\release\` copy, `docs\PROJECT-STATE.md`, passing smoke checks, one commit.

---

## START-GATE — ask before touching anything (AskUserQuestion, 2–4 items)

1. **Python version** — pin 3.12 (recommended: best geopandas/wheels support) vs 3.11 vs latest 3.13?
2. **Scaffold style** — minimal hand-rolled Vite `react-ts` template (recommended) vs create with ESLint/Prettier config now?
3. **README timing** — stub README now at phase 01 (recommended, gives the repo a landing page) vs write it fully in phase 05?
4. **Data copy scope** — all 6 release files verbatim into `data\release\` (recommended — folder is ground truth) vs only the 4 data files?

Log answers in `00-PROJECT-STATE.md` §6 before proceeding. Never re-ask §2 locked decisions.

---

## Tasks

- [ ] `git init` in `hackathon\gridlock\`, branch `main`.
- [ ] Python: `uv init` → `uv venv` → add deps: `fastapi uvicorn geopandas shapely pdfplumber geopy pandas pyproj` (+ `httpx` arrives phase 05, skip now). Keep `pyproject.toml` + `uv.lock` committed.
- [ ] Frontend: `npm create vite@latest frontend -- --template react-ts` → `npm i` → add `@tanstack/react-table maplibre-gl zustand` (+ `tailwindcss @tailwindcss/vite` for styling). Commit `package.json` / `package-lock.json`.
- [ ] Directory skeleton (create empty with `.gitkeep` where needed):
  ```
  gridlock\
    backend\            FastAPI app + pipeline scripts (phase 02–03)
    frontend\           Vite app
    data\
      release\          ← copy all 6 files from research\Sperry-Tech-Challenge\ (verbatim, incl. subfolders)
      processed\        canonical CSVs/GeoJSON land here (phase 02)
    docs\               PROJECT-STATE.md copy lands here (STATE UPDATE)
    scripts\            smoke self-checks live here (one per phase: check_01.py …)
  ```
- [ ] Copy `phase-plans\00-PROJECT-STATE.md` → `gridlock\docs\PROJECT-STATE.md`.
- [ ] `.gitignore`: `.venv/`, `node_modules/`, `frontend/dist/`, `__pycache__/`, `.env` — but **do NOT ignore** `data/` (release + processed are committed: offline-first demo needs them).
- [ ] README stub: title Gridlock, one-line description, phase status, link to `docs\PROJECT-STATE.md`.
- [ ] Smoke checks → `scripts\check_01.py`:
  - `python -c "import fastapi, geopandas, shapely, pdfplumber, geopy, pandas, pyproj"` succeeds
  - `data\release\` contains exactly 6 files
  - `npm run build` in `frontend\` succeeds (Hello-World state)
  - `uv run uvicorn --help` runs
- [ ] `git add -A && git commit` — message `Phase 01: foundation (uv + vite scaffold + release data copy)` + attribution trailer.

## Verify (phase exits green only if all pass)
1. `scripts\check_01.py` prints PASS on every assert.
2. `git log --oneline` shows exactly 1 commit.
3. Opening `frontend\` dev server renders the scaffold page.

## STATE UPDATE
Re-read `00-PROJECT-STATE.md` → append §4 changelog row (`T+~1 | 01 | scaffold + data copy committed | START-GATE answers: …`) → log the 4 START-GATE answers in §6 → set Phase Tracker 01 status `done` → copy file to `gridlock\docs\PROJECT-STATE.md` → include in the commit.

## Halt rule
Intent-level drift (something in §1/§2 of 00 conflicts with reality) → stop, ask. Mechanical fixes (typos, path adjustments, version pins within the START-GATE answer) → proceed + changelog note.