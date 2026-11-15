# Status and handoff

**Last updated:** 15 November 2026. This is the living pick-up note: read it first when resuming. Update it at the end of each working session. For the plan itself see [roadmap.md](roadmap.md); for the build spec see [platform-extraction-task.md](platform-extraction-task.md).

**One-line state:** Phase 1 has started. The prototype is frozen and tagged, golden outputs are captured, the `ponta-platform` skeleton is pushed, and a clickable UI prototype exists in its own repo. **No engine code has been moved yet.** The next action is the module split and move.

## Repositories

| Repo | Local path | Remote (GitHub, `cgvgukapps`) | Position |
|---|---|---|---|
| Blueprint (this repo) | `/Users/cicerovieira/PontaPlan` | `ponta-blueprint` | `main`, tag `blueprint-baseline-2026-11-15` |
| Prototype (frozen) | `/Users/cicerovieira/PONTA-PROTOTYPE` | `ponta-prototype` | `main` at `41bd812`, tag `prototype-validation-final` |
| Platform (production) | `/Users/cicerovieira/ponta-platform` (on disk: `Ponta-Platform`) | `ponta-platform` | `main` at `9443109` (preflight `e6a62c1`, then skeleton) |
| UI prototype | `/Users/cicerovieira/Ponta_Platform_UI` | `ponta-platform-ui` | `main` at `4bd8e52`, tag `prototype-v0.1` |
| Marketing website | `/Users/cicerovieira/Ponta-Website` | `ponta-website` | `main`, v2 live, still `noindex` |

## Done

- **Strategy pivot recorded** across the blueprint: one AI-native product, data tiers, AI layer, Parity Report, roadmap rewritten.
- **Website v2** rebuilt and on `main`. Final hero: "Tactical & physical AI that sees the whole pitch, and the athlete inside it." Sign in disabled, `noindex` on.
- **Frontend design brief v1** written: [frontend-design-brief-v1.md](frontend-design-brief-v1.md). An external agent built a clickable prototype from it (Vite, React, TypeScript, mock data only).
- **Prototype preflight:** 964 tests pass (0 skipped, with network). Tagged `prototype-validation-final`.
- **Golden outputs captured** for Metrica Sample Game 3 and SkillCorner match 1886347. See `ponta-platform/tools/golden/README.md`. Data is git-ignored and regenerable (about 10 minutes).
- **`ponta-platform` skeleton:** two installable packages (empty), pinned requirements, import-graph checker with tests, GitHub Actions CI, README, `docs/ARCHITECTURE.md`. Ruff and 9 tests pass locally.
- **Version control:** all repos above are under git with remotes. The blueprint had none before.
- **Analyses delivered, no files changed:** SoccerTrack dataset (useful for positional fusion validation only: no physiological channels, unclear licence, small); multi-provider wearable integration (see "Strategy gaps" below); hosting options.

## Findings that change the plan

1. **`roster/` belongs in `plaiq_engine`, not `matrix_engine`.** Matrix never imports it, and it imports Plaiq's `DEFAULT_PROFILE`. `pitch_config` belongs in `matrix_engine` (both engines import it). `roster.build_roster` lazily imports `replay.load_game` (a matplotlib demo, not ported) and needs a local CSV loader. Recorded in `ponta-platform/docs/ARCHITECTURE.md`.
2. **`compute_event_pov` row order is non-deterministic across runs; values are deterministic.** Golden `pov.parquet` is stored sorted by `(event_idx, team, jersey)`. The semantic builder must sort explicitly.
3. **The ball is undetected in 32.2% of Metrica Game 3 frames and 26.4% of the SkillCorner match.** The prototype returns an empty control grid there. The Matrix path must match; five ball-missing frames per match are recorded as edge cases.
4. **Decision quality ranges up to 1.5** in both golden matches (minimum 0.0), so it is not a 0 to 1 ratio. Check against the rating bands (Optimal at or above 0.95) before the UI or AI layer assumes a 0 to 1 scale.
5. **SkillCorner event ids are inconsistent** (`From` is `'1'`, `To` is `'1.0'`). Normalise in the new code.
6. **Ruff 0.16.1's default rules are strict.** The prototype has no ruff config, so expect a batch of lint fixes when its code moves in.
7. **UI prototype:** Vite and React Router, not Next.js. Contains `src/lib/engine.ts`, a fake in-browser engine, plus 1.8 MB of synthetic tracking in `src/mock/packs`; neither may reach a production bundle. Its colour tokens diverged from the brief (away is teal, home is `#d6293d`). Its types are hand-written and must become generated types from the API schema.

## Next steps, in order

### A. Finish the extraction (`platform-extraction-task.md`, steps 2 to 7)

1. **Split and move the code** (imports rewritten only, no maths changes):
   - `matrix_engine`: `ingestion/{metrica_io, epts_io, metrica_velocities, skillcorner_io, second_spectrum_io, genius_events_io}`, the existing `matrix_engine/*` modules (`confidence`, `export`, `fusion`, `identity`, `quality`, `schema`, `sync`, `providers`, `testing`), and `pitch_config`.
   - `plaiq_engine`: `analytics/*`, `statsbomb/*`, `roster/*`, `ingestion/statsbomb_io.py`.
   - Not ported (stay in the prototype): `app.py`, `replay.py`, `preprocess.py`.
   - Resolve the unused `prev_event` parameter of `resolve_identities` (remove with call sites, or keep deliberately, and say which).
   - Git history for moved files: decided to skip (`git filter-repo` not installed; full history lives in the prototype at the tag). Revisit only if wanted.
2. **Carry all tests over.** None deleted, skipped or weakened. Report counts before and after, by package. Tests find sample data relative to the repo root: add a resolver that reads `PONTA_SAMPLE_DATA`, defaulting to `../PONTA-PROTOTYPE/sample-data`. Do not copy the data into the repo (licence terms for Metrica and SkillCorner are unresolved, gate G3). The STATSports tests fetch a 51 MB unlicensed export to a pytest temp directory; that data must never enter the repo.
3. **Decide how CI gets the sample data,** or accept that data-dependent tests are gated by the local full run. CI is currently unverified (see gotchas).
4. **Add the facades:** `MatrixPipeline.run()` over the existing flow (`providers → sync → identity → sync(refine) → fusion → confidence → quality → export`), and `MatchAnalysis` over the roughly 30 functions `app.py` imports (inventory them first).
5. **Plaiq-via-Matrix equivalence test** against the golden outputs. Set an explicit, justified tolerance; compare on natural keys, never row order; use `np.allclose` for surfaces and report the maximum difference. Characterise every difference beyond tolerance as an `engine_version` decision; do not edit formulas.
6. **`parity.py`:** the Parity Report. Locate the SkillCorner "Analytics Parity" logic in `app.py`; the new code must reproduce its classification exactly (test it). Rules: GPS-only blocks opposition pressure and control; event-only degrades decision quality; low frame rate or high dropout degrades with the measured figure; no physiological source blocks load and fatigue metrics.
7. **`semantic_builder.py`:** writes `semantic_events.parquet` and `trajectories.parquet`. Deterministic with explicit ordering; `physiological_context` is null unless present (never zero-filled); pseudonymous `player_ref`; no names and no raw coordinate arrays; goalkeepers and set pieces `not_scored`.
8. **Demo script** (Metrica and SkillCorner end to end), then write `EXTRACTION_REPORT.md`.

### B. Rest of Phase 1

Supabase (Postgres, RLS, Auth), FastAPI skeleton, object storage, the automated cross-tenant access test for Parquet paths, basic ToS. See [roadmap.md](roadmap.md#phase-1--platform-foundation-and-engine-extraction).

### C. Frontend

1. Review the UI prototype against the v2 checklist at the end of the design brief; produce brief v2.
2. Import into `ponta-platform/apps/web`. The UI repo now has a baseline commit. Subtree if you keep committing there first, otherwise copy and record the source commit. Archive the old repo afterwards.
3. After import: keep the `src/api` function signatures and add an HTTP client behind a mock switch; move the fake engine and mock packs to dev-only; generate TypeScript types from the OpenAPI schema; add CI path filters.
4. The Parity gate is the first screen that can go real (needs only the Parity Report JSON). Library and Add data follow with the upload endpoint; Replay, Brief and Ask need Phase 2.

### D. Running in parallel

- **Track V (validation):** design-partner outreach; the three data asks (one export, an overlap window, a paired match). Optional: write `soccertrack-validation-task.md` and update the Tier 3 gate status in `data-tiers.md` (offered, not requested).
- **Legal gates** in [roadmap.md](roadmap.md#track-l--legal-and-compliance-runs-alongside-each-item-gated-to-a-trigger): basic ToS before the first pilot; LLM sub-processor terms before any real customer data reaches an LLM.
- **Website:** still open are licence checks for Metrica and SkillCorner (G3), lawyer review of the held items (G4), `/security`, `/privacy`, `/terms` pages, the G6 re-decision on fusion wording, and hero artwork that still shows two engines. Cloudflare deploy status has not been confirmed.

## Open decisions

- LLM provider and model (blocks Phase 2).
- Whether event-only analysis ships in the first release.
- Ask Ponta pricing: capped, metered or bundled.
- Hosting: the blueprint says Fly.io or Render; Railway was discussed; the frontend could be static on Cloudflare. No decision made. Recommendation given: stay on managed services, keep services as plain containers, EU regions from the start, do not move to AWS now.
- Vite versus Next.js for the frontend (the UI prototype is Vite; the blueprint says Next.js). Leaning Vite static export.
- Final dark palette, away-team colour, heat gradient, filled versus outlined primary buttons, Plotly versus Canvas for the pitch.
- Public naming of Matrix and Plaiq; the IP-protection call on publishing `ANALYTICS_SPECIFICATION.md`.

## Docs that now disagree with the evidence (reconcile when convenient)

- `architecture.md` and `platform-extraction-task.md` put `roster/` under `matrix_engine`; the evidence says `plaiq_engine` (finding 1).
- `frontend.md` and `architecture.md` say Next.js; the prototype is Vite.
- `visual-design.md` and the brief hold token values the UI prototype changed (home and away colours, status colours).
- Anything that assumes decision quality is a 0 to 1 ratio (finding 4).

## Strategy gaps found but not acted on

The wearable-integration question (OAuth connections, webhooks, PlayerData, Garmin, Apple Watch, Catapult, STATSports Sonra, session-based sync) is **not** in the strategy beyond file upload, Vault-stored credentials and polling. No action requested. If revisited: add OAuth and token-paste connector types, an optional webhook receiver, and a session entity next to matches, all gated behind the Phase 4 health-data legal review. Apple HealthKit is on-device, so it would need a companion app. Vendor access details in the proposal were not verified.

## How to resume

```bash
cd /Users/cicerovieira/ponta-platform
source .venv/bin/activate            # recreate if missing, see README
python tools/check_imports.py && ruff check packages tools tests && pytest -q
```

Prototype tests use the prototype's own interpreter: `cd /Users/cicerovieira/PONTA-PROTOTYPE && PYTHONDONTWRITEBYTECODE=1 .venv/bin/python -m pytest -q -p no:cacheprovider` (about 2 minutes, expect 964 passed). Regenerate golden outputs: see `tools/golden/README.md`.

## Gotchas

- **CI has not been observed on GitHub.** Check the Actions tab on `ponta-platform`.
- **Install both engine packages in one pip command,** so the local packages resolve each other and pip never looks up an internal name on the public index (distribution names are prefixed `ponta-`).
- **Do not commit `*.egg-info`** (now git-ignored).
- **Sandbox:** writes outside the workspace folder need elevated permissions. Pushes to `main` are held for your approval and need a retry with the approval request. Process listing (`ps`) is blocked.
- **Folder case:** the platform folder is `Ponta-Platform` on disk; macOS ignores case, so `ponta-platform` also works. The venv records the capitalised path.
- **Piping a long job through `grep` hides all output until it finishes,** and the shell reports the pipe's exit status, not the job's.
- **Do not touch the prototype** beyond reading it. It is the validation record.
