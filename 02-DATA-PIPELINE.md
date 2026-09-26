# Phase 02 — DATA PIPELINE (T+2 → T+8h)

**Mission:** turn the release folder into the canonical projects table + our own straight-corridor GeoJSON layer, geocoded with confidence labels, and prove it against the starter workbook's golden 6 rows.

**Inputs:** `00-PROJECT-STATE.md` (FIRST) · `data\release\Finding_Real_Locations_Guide.docx` (read its method end-to-end before coding) · `data\release\Projects_Overlaps.xlsx` (golden fixture + export schema) · `data\release\Project Listings\Dominion Energy\2024-2028-2million-and-above-project-descriptions.pdf` (44 pp) · `data\release\Project Listings\Georgia Power\2025 IRP Volume 3 PUBLIC DISCLOSURE.pdf` (668 pp, Table 2) · report §Build Breakdown 1–2 + §Data Sources §3.

**Outputs:** `data\processed\projects.csv` · `data\processed\gazetteer.csv` · `data\processed\gridlock_projects.geojson` · `scripts\check_02.py` (golden test) · QA notes.

---

## START-GATE (2–4 items; log to 00 §6; never re-ask 00 §2)

1. **Ungeocodeable endpoints** — fallback order when no gazetteer hit: county centroid + `unconfirmed` label (recommended) vs drop the project vs manual lat/lon pin by you?
2. **Santee Cooper** — out of scope (not in the release folder; folder = ground truth) (recommended) vs add as a third utility from live SCRTP sources?
3. **Geocode rate discipline** — Nominatim/Overpass politeness: 1 req/s + retries with backoff (recommended) vs batch overnight vs skip live geocoding and rely only on HIFLD/OSM point layers?
4. **QA depth** — automated labels only (recommended at 24h clock) vs automated + you manually eyeball every `unconfirmed` endpoint on a debug map before exit?

---

## Tasks

### A. Read the ground truth first
- [ ] Extract + read `Finding_Real_Locations_Guide.docx` in full. Record in `docs\DATA-NOTES.md` — the guide's method is already known to be:
  - **Center point** = midpoint of the project's two named sub-points; if only one point is located, that point is the center (workbook formula: `IF(ISBLANK(b), a, IF(ISBLANK(a), b, (a+b)/2))`).
  - **Overlap = center-to-center haversine < 25 mi (40,234 m)**; straight-line only, no driving distance.
  - **`time_gap` = |in_service_date_a − in_service_date_b| in days** (exact integer; scoring still uses years — see 00 §2).
  - Verification loop: re-read the PDF description/zone/landmarks for each match → confirmed, or flag **lower-confidence**.
  - Geocode via Overpass template `nwr["power"="substation"]["operator"~"YOUR_UTILITY_NAME",i](SOUTH,WEST,NORTH,EAST);` at overpass-turbo.eu (swap `substation`→`line` for lines), or Nominatim / Open Infrastructure Map; **scale-up tip from the guide: query all of a utility's tagged infrastructure at once → one GeoJSON per utility → filter to the project list.**
  **The guide's method overrides anything guessed here.**
- [ ] Open `Projects_Overlaps.xlsx` and record exact columns (these are the CSV export contract for phase 03):
  - **projects (10 rows × 17 cols):** `project_id, utility, state, project_name, name_a, lat_a, lon_a, name_b, lat_b, lon_b, lat_center, lon_center, in_service_date, overlap_count, overlap_1, overlap_2, overlap_3` — `overlap_N` holds the counterpart **project_id** (not an OVL id).
  - **overlaps (6 rows × 9 cols):** `overlap_id, distance_mi, time_gap (day), utility_a, project_id_a, project_name_a, utility_b, project_id_b, project_name_b`.
  - Dates may be text (`12/31/2024`) or **Excel serials** (`45809` = 2025-06-01, `45778` = 2025-05-01) — parse both.

### B. Ingest (schema below; provenance on every row)
`data\processed\projects.csv`:
```
project_id, name, utility, type(line|substation|plant), endpoint_a, endpoint_b,
voltage_kv, in_service_year(or date), status, cost_usd, county, sponsor,
source_file, source_page, notes
```
- [ ] **DESC:** `pdfplumber` over the release Dominion PDF → 44 rows (one per page: ID, description w/ endpoints, status, Planned In-Service Date, cost). Parse dates → `in_service_year`; keep raw date in notes for `time_gap`. **Cross-check:** the starter includes `DESC_5 Okatie-Bluffton 115 kV: Rebuild` (in-service 2025-06-01) — if the release PDF lacks an Okatie–Bluffton row, record the discrepancy in DATA-NOTES and use the starter row (ground truth), do not silently drop it.
- [ ] **GPC:** `pdfplumber` over release Vol.3 → GA ITS Ten-Year Plan **Table 2**; keep **Sponsor == "GPC" only (≈122 rows)** — others dropped, count logged; columns Zone, TEAMS #, Name, Need Date, Sponsor (costs are REDACTED in source — leave blank). Need Date → `in_service_year`.
- [ ] Hand-transcribe fallback: if a table fights the parser past ~45 min, transcribe rows manually into CSV (report §Build Breakdown 1 explicitly allows this) and note it in DATA-NOTES.

### C. Geocode + own layer (the "create our own layer" answer = YES)
- [ ] Endpoint gazetteer `data\processed\gazetteer.csv`:
  ```
  endpoint_name, state_hint, lat, lon, source, confidence(confirmed|likely|unconfirmed), checked_on
  ```
  **SEED FIRST from the starter workbook (confidence=`confirmed`, source=`starter`)** — 17 named endpoints already carry coordinates: Stevens Creek Sub, Hooks Sub, Thurmond Sub, Jasper Sub, Okatie Sub, Queensboro Sub, Ft Johnson Sub, Bluffton Sub (DESC); EVANS PRIMARY, THURMOND DAM #5, MCINTOSH, PURRYSBURG, GOSHEN, Mitchell Substation, North Tifton Substation, JESUP, LUDOWICI PRIMARY (GPC). Any later live geocode that disagrees with a seeded coordinate is a bug — label `unconfirmed` + note.
  Sources in priority order (all live, exact URLs):
  1. HIFLD substations: `https://services1.arcgis.com/CD5mKowwN6nIaqd8/arcgis/rest/services/project_renewable_us_substations_2022/FeatureServer/10/query` (79,687 pts; bbox-filter SC+GA ≈ 4,019)
  2. OSM Overpass named substations: POST `https://overpass-api.de/api/interpreter`, `way["power"="substation"](bbox); out tags geom;` (≈5,587 in bbox)
  3. EIA 860M plant anchors: `https://www.eia.gov/electricity/data/eia860m/xls/august_generator2026.xlsx` (lat/long incl. Planned units)
  4. Nominatim `https://nominatim.openstreetmap.org/search` (state-filtered; 1 req/s, `User-Agent` header)
  Confidence: exact facility-name hit on layer 1/2 → `confirmed`; town-level → `likely`; county centroid fallback → `unconfirmed`.
- [ ] Build `data\processed\gridlock_projects.geojson` (EPSG:4326): line projects → straight `LineString` between endpoints; substation/plant → `Point`. **Property `geometry_basis: "straight corridor (endpoint geocode)"` on every feature.** Label everywhere: "corridor proximity, not surveyed distance."
- [ ] Layer must also carry `confidence` (worst of its endpoints) for map styling later.

### D. Golden test (the phase gate)
- [ ] `scripts\check_02.py` — **the ±0.1 mi assertion binds to the GUIDE's method: center-point haversine** (NOT closest-point; the engine's metric differs by design and is checked separately below):
  - Load the starter's 10 projects (use its `lat_center`/`lon_center`, or recompute with the midpoint formula — they must agree).
  - Compute center-to-center **haversine** distances; compute `time_gap` = |date_a − date_b| in days.
  - **Assert each of the 6 rows matches within ±0.1 mi, and each time_gap matches exactly** (Excel-serial dates included):

    | overlap | pair | distance_mi | time_gap (day) |
    |---|---|---|---|
    | OVL_1 | DESC_2 × GPC_1 | 4.09 | 3074 |
    | OVL_2 | DESC_3 × GPC_2 | 5.65 | 152 |
    | OVL_3 | DESC_3 × GPC_3 | 7.55 | 517 |
    | OVL_4 | DESC_1 × GPC_1 | 8.01 | 3074 |
    | OVL_5 | DESC_5 × GPC_2 | 14.34 | 365 |
    | OVL_6 | DESC_5 × GPC_3 | 14.81 | 730 |

  - Print expected vs got for all 6.
- [ ] Method-consistency (informational, no starter assert): same center-pairs via `to_crs(EPSG:5070)` must agree with haversine to ≈±0.05 mi; closest-point distances will legitimately differ from the center values — store BOTH (`distance_center_mi` for starter parity, closest-point for the engine) in `data\processed\`.
- [ ] Assert row counts: DESC = 44; GPC-sponsored ≈122 (±tolerance 5, log exact); every projects row has `source_file`.
- [ ] Assert every gazetteer row has a confidence label; assert every GeoJSON feature has `geometry_basis`.
- [ ] Spot cross-check vs live (freshness decision): re-fetch ONE source (e.g. HIFLD query count) and note drift vs release in DATA-NOTES.

## Verify
1. `uv run python scripts\check_02.py` → PASS (golden 6 rows ±0.1 mi is non-negotiable; fix geocoding until green).
2. Eyeball: render a quick debug map (folium one-off) of corridors + halos over SC/GA border; no endpoint in the ocean or wrong state.
3. `docs\DATA-NOTES.md` contains guide method + xlsx exact columns.

## STATE UPDATE
00 §4 changelog row (`T+~7 | 02 | projects.csv + geojson + gazetteer; golden 6/6 pass`) · START-GATE answers → §6 · tracker status · sync to `gridlock\docs\`.

## Halt rule
If the guide or workbook contradicts a §2 locked decision (e.g. different golden tolerance) → halt and ask; do not silently reinterpret. Parser quirks, path issues, retry tuning → proceed + changelog.