# Phase 04 — FRONTEND (UI Example 3 / Concept C "Ledger") (T+12 → T+19h)

**Mission:** the locked UI — sortable ranked coordination table as the primary surface, sticky linked map inset, palette-bound, with AI slots stubbed for phase 05.

**Inputs:** `00-PROJECT-STATE.md` (FIRST) · report §UI Examples **Concept C** + "Recommended row schema" + color language (`research\gridlock-evaluation.md`) · `/api/*` from phase 03 · **skill `nextlevelbuilder/ui-ux-pro-max-skill`** (invoke at task A1 — user directive).

**Outputs:** built React app (dev state), two-way table↔map sync, CSV export button, command bar + detail drawer slots (empty, wired), `scripts\check_04` (build smoke).

---

## START-GATE (2–4 items; log to 00 §6)

1. **Light basemap tiles:** free no-key options — Carto Positron raster tiles (recommended: nicest light look, needs ODbL attribution) vs MapLibre demo tiles (ugliest, most reliable) vs self-hosted OSM tile proxy (most work)?
2. **Expanded rows:** excluded/behind-top-10 rows — show first 50 + "load more" (recommended) vs virtualize all (TanStack virtual) vs paginate 25/page?
3. **Mobile:** desktop-demo-only acceptable (recommended — judging on laptops) vs basic responsive stack (table → cards under 768px)?
4. **Empty state:** if zero overlaps survive, show "no pairs ≤40 km" explainer with nearest-candidates list (recommended) vs plain empty table?

---

## Tasks

### A. Design system (skill first, palette governs)
- [ ] **Invoke skill `nextlevelbuilder/ui-ux-pro-max-skill`** → use its `design-system` guidance to set up tokens/typography/spacing for the app, **building AROUND the report palette** — the skill's job is craft; these hues are non-negotiable:
  - Utilities: DESC `#2563EB`, GPC `#059669`
  - Halos: touching `#DC2626`, <1.6 km `#F97316`, <8 km `#FBBF24`, <40 km `#FDE68A`
  - Excluded: grey @ 20% opacity. **Legend always visible.** Basemap: light only.

### B. Layout — Concept C wireframe (from the report; placeholder data shown):
```text
+------------------------------------------------------------------+----------------------------------+
| GRIDLOCK  DESC x GPC  [tiers: all|touch|<1.6|<8|<40] [window: any] | MAP (sticky inset, ~40%)        |
+------------------------------------------------------------------+----------------------------------+
| RANKED COORDINATION OPPORTUNITIES   n=…   [export csv]            |                                  |
| # | Pair                    | Tier | Dist | Window | Score | Est.$ | |  corridors + halos, flyTo       |
| 1 | <desc pair>             | <1.6 | 0.4km |  2026  |  92  |  --  | |  DESC blue / GPC green lines    |
| ... top 10 ...               |      |       |        |       |       | |                                  |
| — excluded rows, greyed, expandable below —                     |  [+/-] [layers] [fullscreen]     |
+---------------------------+------+-------+-------+-------+-------+--------------------------------+
| DETAIL DRAWER (row click; expands under selection, full width)                                   |
|  pair summary | closest-point segment | years/time_gap_days | AI PAIR BRIEF slot (phase 05)        |
+------------------------------------------------------------------------------------------+
| COMMAND BAR (fixed): [Ask about the table… ____________]  [status chip slot (phase 05)]          |
+------------------------------------------------------------------------------------------+
```
- [ ] Stack: React + TS; **TanStack Table** for the ledger (columns per report schema: `rank, project_a, project_b, utilities, min_distance_km, tier, shared_in_service_year, time_gap_days, score, est_shared_row_acres(null for now)`); default sort score desc; tier/window filter chips re-render + count badge; top 10 shown, rest (incl. greyed excluded) per START-GATE 2.
- [ ] **MapLibre inset (sticky ~40%):** GeoJSON sources from `/api/projects` + `/api/overlaps` (shortest-line segments as halo strokes); line layers in utility hues; tier halo colors exact hexes; excluded greyed 20%; light basemap per START-GATE 1; always-on legend (tier swatches + utility hues); fullscreen "Expand map" toggle for the required pan/zoom/click demo moment.
- [ ] **Two-way sync (Zustand single store):** row hover → inset `flyTo` pair bbox + halo glow; row click → detail drawer opens + pulse-lock; halo click → row scroll-into-view + highlight; Escape closes drawer.
- [ ] **CSV export button** → `GET /api/export/overlaps.csv` (browser download).
- [ ] **Slots stubbed now, filled in 05:** command bar input (reserve layout space now), detail-drawer "Pair brief" section, status-chip container in the command bar.
- [ ] Honesty labels in UI footer: "corridor proximity — straight-line approximation from geocoded endpoints, not surveyed routes"; ODbL/attribution for OSM-derived data.

## Verify
1. `npm run build` succeeds (`scripts\check_04`: build + grep built assets for all 6 palette hexes — they must appear).
2. Manual pass (checklist in commit message): hover row → map moves; click halo → row highlights; sort every column; tier filter updates count badge; excluded rows greyed below top 10; export CSV opens with starter-matching headers; legend visible at all zooms; fullscreen map works.
3. Colors: visual check against hexes (no "close enough" blues/greens).

## STATE UPDATE
00 §4 (`T+~18 | 04 | Ledger UI + linked inset live; skill applied; palette verified`) · §6 answers · tracker · sync to `gridlock\docs\`.

## Halt rule
Any proposal to change the primary surface (e.g. "map would impress judges more") = intent-level (UI Example 3 locked) → halt + ask. Styling refinement, spacing, animation = mechanical, proceed + changelog.