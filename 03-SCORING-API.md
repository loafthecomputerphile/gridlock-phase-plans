# Phase 03 — SCORING ENGINE + API (T+8 → T+12h)

**Mission:** the canonical overlap engine (EPSG:5070, tiers, window bonus, tie-break) behind a FastAPI service, proven by the golden test, with starter-schema CSV export.

**Inputs:** `00-PROJECT-STATE.md` (FIRST) · `data\processed\projects.csv` + `gridlock_projects.geojson` + `gazetteer.csv` (phase 02) · `data\release\Projects_Overlaps.xlsx` (export contract) · report §Build Breakdown 3–6 (verbatim method).

**Outputs:** `backend\engine.py` · `backend\app.py` (FastAPI) · `scripts\check_03.py` · `data\processed\overlaps.csv`.

---

## START-GATE (2–4 items; log to 00 §6)

1. **Excluded-pair rows (>40 km):** the UI shows excluded rows greyed — how many? A capped exemplar set, e.g. the 25 nearest excluded pairs (recommended) vs all computed pairs (explodes rows) vs none (violates the greyed-rows decision)?
2. **CSV export scope:** all rows returned by the API incl. excluded exemplars (recommended — "CSV exports all" refers to what the app shows) vs every pair ever computed?
3. **25 mi vs 40 km edge:** the outline/guide flag pairs < 25 mi (40,234 m); the locked tier rule includes ≤ 40,000 m. Pairs landing in (40,000, 40,234] m — exclude per the locked 40 km rule (recommended — 00 §2 governs; no starter row falls in this sliver) vs include as `<40 km` per the outline's literal 25 mi?
4. **Date → window edge:** starter rows mix text dates and Excel serials (45809 = 2025-06-01): derive the build window from the YEAR only `[2026, 2027]` (recommended — canonical rule is year-based) vs month-aware windows?

---

## Tasks

### A. Engine (`backend\engine.py`) — canonical rule, byte-identical wording in the docstring:
> score = tier points (crossing=100, <1.6 km=60, <8 km=40, <40 km=20) + 15 if build windows overlap, total capped at 100; tie-break by smaller closest-point distance

- [ ] Load geojson → `GeoDataFrame` → **`to_crs(5070)` before ANY distance math. Never compute in degrees.**
- [ ] Candidate prefilter: `gdf_a.sjoin(gdf_b, predicate="dwithin", distance=40000)` (or shapely 2 `STRtree.query(..., predicate="dwithin", distance=40000)`); **then always re-check exact `a.distance(b) ≤ 40000`** (buffer/dwithin boundary artifacts).
- [ ] Exact metric per pair: `d = a.distance(b)` (m) — closest-point distance for any geometry mix; `shapely.ops.shortest_line(a, b)` for the map segment (export as geometry).
- [ ] Tier function (strict `<`, inclusion `≤ 40 km`):

  | condition | tier | points |
  |---|---|---|
  | `d == 0` / `a.intersects(b)` | touching/crossing | 100 |
  | `d < 1600` | `<1.6 km` | 60 |
  | `d < 8000` | `<8 km` | 40 |
  | `d ≤ 40000` | `<40 km` | 20 |
  | `d > 40000` | excluded | 0 |

- [ ] Timeline: `build_window = [in_service_year, in_service_year + 1]`; overlap test `a0 <= b1 and b0 <= a1`; +15 else +0; `score = min(100, tier_points + bonus)`. Year missing → bonus 0 + `year_unknown: true` flag (never drop the row).
- [ ] Sort: score desc, tie-break distance asc. Keep `tier`, `distance_m`, years as separate fields — judges must see WHY (report invariant: far-but-same-year never outranks crossing).
- [ ] Also compute, per pair: **`time_gap` = |in_service_date_a − in_service_date_b| in days** (exact integer, Excel-serial aware — display/QA per the guide; scoring uses years) and **`distance_center_mi`** (center-point haversine, starter-parity column for golden QA). Pull exact semantics from `docs\DATA-NOTES.md`. The UI/rank distance remains closest-point EPSG:5070 (00 §2).

### B. API (`backend\app.py`)
- [ ] `GET /api/health` → `{status, phases_done, data_files}`
- [ ] `GET /api/projects` → projects.csv rows + geometry basis/confidence
- [ ] `GET /api/overlaps?tier=&sort=score|distance|year&limit=` → ranked pairs (incl. excluded exemplars per START-GATE 1), each: `rank, project_a, project_b, utilities, min_distance_km, distance_center_mi, tier, shared_in_service_year, time_gap, score, shortest_line geometry, year_unknown`
- [ ] `GET /api/pairs/{pair_id}` → full detail (for detail drawer)
- [ ] `GET /api/export/overlaps.csv` → **column names identical to the starter workbook's overlaps sheet** (contract captured in DATA-NOTES phase 02); all rows per START-GATE 2
- [ ] Pydantic response models; CORS for the Vite dev origin (`http://localhost:5173`).

### C. Golden test as API test
- [ ] `scripts\check_03.py`: boot the app (TestClient), hit `/api/overlaps`, and **re-assert the starter's 6 rows ±0.1 mi through the API path** (not just the engine). Assert tier boundary unit cases: d=1599→`<1.6`, d=1600→`<8`, d=8000→`<8`, d=40000→`<40`, d=40001→excluded; assert crossing+no-window score 100 (cap), `<40`+window 35; assert tie-break order; assert CSV header == starter header.

## Verify
1. `uv run python scripts\check_03.py` → PASS.
2. `uv run uvicorn backend.app:app` serves `/api/health` 200; export downloads and opens in a spreadsheet.
3. Invariant spot-check printed: no far-same-year pair above any crossing pair.

## STATE UPDATE
00 §4 (`T+~11 | 03 | engine + API; golden 6/6 via API; boundaries green`) · §6 answers · tracker · sync to `gridlock\docs\`.

## Halt rule
Any temptation to change tier points, window rule, cap, or tie-break = intent-level → halt + ask. Endpoint shape tweaks, limit defaults, error formats → mechanical, proceed + changelog.