# Phase 06 — STRETCH (T+23 → T+24h+; every item here is cut-first)

**Mission:** the optional layers — only reachable after phase 05 verifies green and the ship floor is intact. **Nothing here may regress phases 01–05.** If the clock runs out, ship what exists and stop.

**Inputs:** `00-PROJECT-STATE.md` (FIRST) · `/api/overlaps` + engine (03) · UI (04) · report §MVP vs stretch + bonus methods + acre math.

**Outputs (any subset):** bonus cost/impact column + panel · snap-to-existing geometry variant · pitch deck + 90s script · README polish.

---

## START-GATE (2–4 items; log to 00 §6)

1. **Priority if only one lands:** pitch deck + 90s script (recommended — judged event needs the pitch) vs bonus cost/impact column vs snap-to-existing layer?
2. **Bonus method** (report offers both): shared-ROW acres → $ (recommended — DESC costs are real, GPC REDACTED, acre math is transparent) vs crew-mobilization estimate ($150–300k per coordinated site <8 km) vs show both with a toggle?
3. **Pitch format:** 5-slide outline + 90-second script (recommended) vs 3-slide ultra-lean vs 10-slide full story?
4. **Snap-to-existing scope:** snap endpoint pairs to nearest HIFLD/OSM line within X km as an OPTIONAL geometry toggle (recommended, X=2 km) vs skip entirely?

---

## Tasks

### A. Bonus cost/impact (pick method per START-GATE 2)
- [ ] Engine: `est_shared_row_acres` from closest-approach segment length × ROW width. **Acre math (exact):** `width_ft × 5,280 ÷ 43,560`; typical ROW 100–150 ft for ≥230 kV ≈ **12–18 acres/mile**. Expose the width input as a visible UI control; label every dollar **"illustrative assumption"** with inputs shown. $/acre or mobilization figure from START-GATE answer.
- [ ] DESC side only where costs exist (release DESC PDF has year-by-year costs; GPC = REDACTED — show `--` + footnote, never invent).
- [ ] UI: `Est.$` column fills in; detail drawer shows assumption table (width, length, $/unit, multiplier).
- [ ] Golden check: `scripts\check_06_bonus.py` — one known pair recomputes by hand within 1%.

### B. Snap-to-existing (optional geometry variant)
- [ ] For each endpoint pair, query HIFLD lines FeatureServer (`https://services2.arcgis.com/LYMgRMwHfrWWEg3s/arcgis/rest/services/HIFLD_US_Electric_Power_Transmission_Lines/FeatureServer`, bbox-filter SC+GA ≈ 4,043) + OSM Overpass lines in bbox; if an existing line lies within START-GATE 4's distance, offer `geometry_variant: "snapped"` alongside `"straight"`.
- [ ] Variant toggle in the map layer switcher; **default stays `straight`** (golden distances must keep passing on straight); re-run `check_02`/`check_03` after enabling to prove no regression.

### C. Pitch + demo polish
- [ ] `docs\PITCH.md`: 5-slide outline (problem → data method → overlap engine + golden proof → Ledger demo beat → Sperry-AI framing) + 90s script that runs in ≤90s (time it). Reuse README's Sperry paragraph; mention the intern listing from `data\release\Opportunities\`.
- [ ] Demo path walkthrough: one rehearsed click-path (open → hover top row → drawer → brief from cache → fullscreen map → NL query example) — write it as a checklist at the top of PITCH.md.
- [ ] README final pass: any stretch items actually shipped get documented; uns shipped ones explicitly listed as "cut" (honesty over feature count).

## Verify
1. Ship floor re-check: run `check_01`…`check_05` scripts — **all still PASS** (stretch never regresses).
2. Any stretch feature that landed has its own assert in `scripts\check_06*.py`.
3. PITCH.md read-aloud ≤ 90 s.

## STATE UPDATE
00 §4 (`T+~24 | 06 | stretch: <list what actually landed / cut>`) · §6 answers · tracker: mark 06 `done/partial/cut` honestly · sync to `gridlock\docs\` · final commit if anything landed.

## Halt rule
Stretch is OPTIONAL — if any item threatens the ship floor or the 24h clock, cut it and record the cut (mechanical, no ask needed — cut-first was pre-authorized). New scope during stretch (new features, new data sources, model swaps) = halt + ask.