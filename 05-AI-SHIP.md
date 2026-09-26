# Phase 05 — AI INTEGRATION + SHIP (T+19 → T+23h)

**Mission:** pair-brief generator + NL table query behind the provider adapter with cache and graceful fallback; single-port offline demo; README; full smoke pass. **This phase completes the ship floor (map + table + 1 AI).**

**Inputs:** `00-PROJECT-STATE.md` (FIRST) · `/api/pairs/{id}` + `/api/overlaps` (phase 03) · UI slots from phase 04 · report §CEII red line + honesty caveats (README content) · `data\release\Software_Engineer_Intern_Listing.docx` + Sperry framing (README AI paragraph).

**Outputs:** `backend\ai\` (adapter, prompts, cache) · wired UI (command bar, drawer, chip) · static build served by FastAPI · `README.md` · `scripts\check_05.py`.

---

## START-GATE (2–4 items; log to 00 §6)

1. **API key handling:** `OPENROUTER_API_KEY` via `.env`/env-var, app runs degraded without it (recommended — judge room may lack net/key, cache covers demo) vs hardcoded key (never) vs no key at all (AI feature dead)?
2. **Cache pre-warm:** pre-generate pair briefs for all top-10 pairs while you still have a connection (recommended — guarantees the offline demo) vs lazy on first click?
3. **NL query visibility:** show the AI's parsed query as chips above the table ("tier = crossing AND year = 2026") before applying (recommended — judges see the AI working) vs apply silently and show only results?
4. **Brief regeneration:** cached briefs are permanent for the run (recommended — deterministic demo) vs TTL 10 min + regenerate button?

---

## Tasks

### A. Provider adapter (`backend\ai\adapter.py`)
- [ ] OpenRouter: `POST https://openrouter.ai/api/v1/chat/completions`, model pinned **`meta-llama/llama-3.3-70b-instruct:free`**; **fallback-on-error** (429/5xx/timeout) → secondary free model in the same call chain, then → structured `{status: unavailable}` — never a stack trace to the UI.
- [ ] Adapter interface kept provider-neutral (swap later: same `complete(prompt, schema) -> result` signature — Sperry/other providers noted as future).
- [ ] `httpx` with 15 s timeout; retries: 2 with backoff for 429 only.
- [ ] **Disk JSON cache** `data\processed\ai_cache\` — key = sha256(model + prompt_version + payload); write-through on success; read-first on every call. `prompt_version` constant bumped when prompts change.

### B. Feature 1 — pair brief (ships this phase; ship floor = this)
- [ ] `POST /api/ai/brief {pair_id}` → brief via drawer. **Planner-prose: 3–4 sentences.** Prompt template injects ONLY pair facts (names, utility hues, distance, tier, score, windows, `time_gap_days`, confidence, geometry_basis) + instruction: professional planner voice, no invented numbers, no advice beyond coordination framing.
- [ ] Response `{text, source: live|cached, model, status}`. UI: detail drawer "Pair brief" section with status chip + Retry button; states: live / cached / rate-limited / unavailable (chip colors distinct from tier palette).

### C. Feature 2 — NL table query (filter/sort/count ONLY)
- [ ] `POST /api/ai/query {text}` → model returns STRICT JSON `{filters:[{col,op,value}], sort:{col,dir}, intent:"count|filter"}` against a whitelisted column set (tier, score, min_distance_km, utility, shared_in_service_year, year…). Validate server-side; reject anything outside whitelist (no free-form SQL/JS — ever).
- [ ] Apply to `/api/overlaps` server-side; respond `{parsed, row_count, rows}`.
- [ ] **Fallback when AI down:** local keyword parser (same whitelist: "crossing/tier<8/2026/score>60…") → same response shape with `source: local`. UI command bar shows chip (phase-04 slot); per START-GATE 3 show parsed chips above the table.

### D. Single-port offline ship
- [ ] `npm run build` → FastAPI mounts `frontend/dist` at `/` (StaticFiles + SPA fallback to index.html); `/api/*` unchanged. One command demo: `uv run uvicorn backend.app:app --port 8000`.
- [ ] Offline proof: airplane-mode/devtools-offline → app loads, table+map work, briefs served `cached`, NL served `local`, chips honest.

### E. README (`gridlock\README.md`)
- [ ] What it is + 30-second demo script (start command, what to click).
- [ ] Data provenance (release folder, per-file), **CEII statement** (public files only; no exact critical-asset routes), **straight-corridor honesty caveat** ("corridor proximity, not surveyed distance"), scoring rule paragraph (canonical wording), live cross-check spot note (freshness).
- [ ] **Sperry Tech AI framing paragraph** (user-locked, cite specifics from the release intern listing `data\release\Opportunities\Software_Engineer_Intern_Listing.docx`): Sperry Tech's **AI Department "builds software and machine learning capabilities for the business"** for its construction parent — extraction pipelines from business systems, **validation and data-quality checks**, training-data prep, **document parsing and metadata extraction**. This app mirrors that exact stack against utility filings: PDF extraction → validation against the starter's golden fixture → AI-written coordination briefs — precisely the challenge's ask to integrate AI.

## Verify
1. `uv run python scripts\check_05.py`: adapter unit (mocked 200/429/timeout → live/fallback/unavailable paths); cache hit on second brief call with network blocked; NL query happy-path + malicious prompt ("drop table" / free-form question) → rejected or local-filtered, never executed; `/` serves built index.html; `/api/health` still 200.
2. Full manual: demo on offline network — every chip state reachable and truthful; Retry works when key present.
3. Ship-floor check: map ✓ table ✓ pair briefs ✓ — only THEN mark phase 05 done.

## STATE UPDATE
00 §4 (`T+~22 | 05 | AI adapter + briefs + NL query; offline single-port ship; README`) · §6 answers · tracker · sync to `gridlock\docs\`.

## Halt rule
Adding AI features beyond the two locked ones, switching models without asking, or changing ship-floor order = halt + ask. Timeout tuning, cache path fixes, chip copy = mechanical + changelog.