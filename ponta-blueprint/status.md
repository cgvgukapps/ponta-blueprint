# Status and handoff

**Last updated:** 15 November 2026. This is the living pick-up note: read it first when resuming. Update it at the end of each working session. For the plan itself see [roadmap.md](roadmap.md); for the build spec see [platform-extraction-task.md](platform-extraction-task.md).

**One-line state:** Phase 1 is under way. The prototype is frozen and tagged, golden outputs are captured, and the engine code has been moved into `ponta-platform` (commit `8603dd7`), with lint clean and CI green (`36ef7d8`, 15 November 2026). 973 tests pass locally. The next action is the facades, the equivalence test, `parity.py` and `semantic_builder.py`.

## Repositories

| Repo | Local path | Remote (GitHub, `cgvgukapps`) | Position |
|---|---|---|---|
| Blueprint (this repo) | `/Users/cicerovieira/PontaPlan` | `ponta-blueprint` | `main`, tag `blueprint-baseline-2026-11-15` |
| Prototype (frozen) | `/Users/cicerovieira/PONTA-PROTOTYPE` | `ponta-prototype` | `main` at `41bd812`, tag `prototype-validation-final` |
| Platform (production) | `/Users/cicerovieira/ponta-platform` (on disk: `Ponta-Platform`) | `ponta-platform` | `main` at `36ef7d8` (move `8603dd7`, then lint and CI-skip changes) |
| UI prototype | `/Users/cicerovieira/Ponta_Platform_UI` | `ponta-platform-ui` | `main` at `a2d3e62` (design-review changes, not pushed); tag `prototype-v0.1` at `4bd8e52` |
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

**Progress, 15 November 2026 (pushed to `ponta-platform` `main`, head `36ef7d8`):** items 1 and 2 below are done apart from the points listed here. Import-graph check passes. 973 tests pass (964 carried from the prototype plus the 9 skeleton tests), none deleted or weakened. By location: `matrix_engine` 245, `plaiq_engine` 673, repo-level 55 (46 cross-engine, 9 import-graph). Layout: `matrix_engine/ingestion/` and `matrix_engine/pitch_config.py`; `plaiq_engine/{analytics,statsbomb,roster,ingestion/statsbomb_io}`. Added `matrix_engine/sample_data.py` (resolver: `PONTA_SAMPLE_DATA`, default `../PONTA-PROTOTYPE/sample-data`) and a local CSV loader in `roster/build_roster.py` in place of `replay.load_game`. `test_skillcorner_io.py` and `test_ss_roster.py` sit in `tests/cross_engine/` because they import `roster`; shared fixture builders are in `tests/support/`. The three path checks in `test_statsports_provider.py` now use the real repo root, so the "nothing from this dataset in the repo" scans still cover the whole repo. **Lint:** safe ruff fixes applied; the remaining style rules are ignored in `ruff.toml` with a note (not endorsed; remove entries when the code is next changed). **CI (item 3, decided):** CI is green. Data-dependent tests skip when the sample data is absent (CI: 841 passed, 132 skipped, by the local no-data run; the exact CI counts were not read). The full run is the local gate: `PONTA_REQUIRE_SAMPLE_DATA=1 pytest -q` expects 973 passed, 0 skipped. Revisit when gate G3 (Metrica and SkillCorner licences) is settled. **Still open:** `prev_event` in `resolve_identities` (not touched yet).

1. **Split and move the code** (imports rewritten only, no maths changes):
   - `matrix_engine`: `ingestion/{metrica_io, epts_io, metrica_velocities, skillcorner_io, second_spectrum_io, genius_events_io}`, the existing `matrix_engine/*` modules (`confidence`, `export`, `fusion`, `identity`, `quality`, `schema`, `sync`, `providers`, `testing`), and `pitch_config`.
   - `plaiq_engine`: `analytics/*`, `statsbomb/*`, `roster/*`, `ingestion/statsbomb_io.py`.
   - Not ported (stay in the prototype): `app.py`, `replay.py`, `preprocess.py`.
   - Resolve the unused `prev_event` parameter of `resolve_identities` (remove with call sites, or keep deliberately, and say which).
   - Git history for moved files: decided to skip (`git filter-repo` not installed; full history lives in the prototype at the tag). Revisit only if wanted.
2. **Carry all tests over.** None deleted, skipped or weakened. Report counts before and after, by package. Tests find sample data relative to the repo root: add a resolver that reads `PONTA_SAMPLE_DATA`, defaulting to `../PONTA-PROTOTYPE/sample-data`. Do not copy the data into the repo (licence terms for Metrica and SkillCorner are unresolved, gate G3). The STATSports tests fetch a 51 MB unlicensed export to a pytest temp directory; that data must never enter the repo.
3. ~~Decide how CI gets the sample data.~~ Done: skip in CI, full run is the local gate (see Progress above).
4. **Add the facades:** `MatrixPipeline.run()` over the existing flow (`providers → sync → identity → sync(refine) → fusion → confidence → quality → export`), and `MatchAnalysis` over the roughly 30 functions `app.py` imports (inventory them first).
5. **Plaiq-via-Matrix equivalence test** against the golden outputs. Set an explicit, justified tolerance; compare on natural keys, never row order; use `np.allclose` for surfaces and report the maximum difference. Characterise every difference beyond tolerance as an `engine_version` decision; do not edit formulas.
6. **`parity.py`:** the Parity Report. Locate the SkillCorner "Analytics Parity" logic in `app.py`; the new code must reproduce its classification exactly (test it). Rules: GPS-only blocks opposition pressure and control; event-only degrades decision quality; low frame rate or high dropout degrades with the measured figure; no physiological source blocks load and fatigue metrics.
7. **`semantic_builder.py`:** writes `semantic_events.parquet` and `trajectories.parquet`. Deterministic with explicit ordering; `physiological_context` is null unless present (never zero-filled); pseudonymous `player_ref`; no names and no raw coordinate arrays; goalkeepers and set pieces `not_scored`.
8. **Demo script** (Metrica and SkillCorner end to end), then write `EXTRACTION_REPORT.md`.

### B. Rest of Phase 1

Supabase (Postgres, RLS, Auth), FastAPI skeleton, object storage, the automated cross-tenant access test for Parquet paths, basic ToS. See [roadmap.md](roadmap.md#phase-1--platform-foundation-and-engine-extraction).

### C. Frontend

1. ~~Review the UI prototype against the v2 checklist.~~ Done 15 November 2026, and the changes are made in the UI repo (`a2d3e62`, see its `docs/design-notes.md` section 6): engine DQ bands and the 0 to 1.5 scale, citation chips that lead with the moment, "Data check" and source-based tier names, labelled parity counts, calmer replay overlays. Still to do: fold the decisions into a brief v2, and add the coverage figure to the data check once the engine produces it.
2. Import into `ponta-platform/apps/web`. The UI repo now has a baseline commit. Subtree if you keep committing there first, otherwise copy and record the source commit. Archive the old repo afterwards.
3. After import: keep the `src/api` function signatures and add an HTTP client behind a mock switch; move the fake engine and mock packs to dev-only; generate TypeScript types from the OpenAPI schema; add CI path filters.
4. The Parity gate is the first screen that can go real (needs only the Parity Report JSON). Library and Add data follow with the upload endpoint; Replay, Brief and Ask need Phase 2.

### D. Running in parallel

- **Track V (validation):** design-partner outreach; the three data asks (one export, an overlap window, a paired match). Optional: write `soccertrack-validation-task.md` and update the Tier 3 gate status in `data-tiers.md` (offered, not requested).
- **Legal gates** in [roadmap.md](roadmap.md#track-l--legal-and-compliance-runs-alongside-each-item-gated-to-a-trigger): basic ToS before the first pilot; LLM sub-processor terms before any real customer data reaches an LLM.
- **Website:** still open are licence checks for Metrica and SkillCorner (G3), lawyer review of the held items (G4), `/security`, `/privacy`, `/terms` pages, the G6 re-decision on fusion wording, and hero artwork that still shows two engines. Cloudflare deploy confirmed 15 November 2026: pontahq.com returns 200 with `noindex, nofollow`.

## Open decisions

- **Provider list on the website:** the v2 page lists Second Spectrum as supported; the engine record has it fixture-tested only and the hard rule allows four providers. Change the page or revise the rule with a reason.
- **Evidence-gate thresholds (E1 to E3)** in [roadmap.md](roadmap.md#evidence-gates-proposed-15-november-2026-founder-to-confirm-the-thresholds): confirm or change.
- **Lead wedge** (single-export brief or fragmentation and provider transition): decided by discovery calls, see [go-to-market.md](go-to-market.md#two-wedges-one-to-lead-with-added-15-november-2026).
- **Facade decisions** (input form for `MatchAnalysis`, rendering helpers, private-name wrappers): parked pending the strategy review; recommendations are in `ponta-platform/docs/MATCH_ANALYSIS_INVENTORY.md`.
- LLM provider and model (blocks Phase 2).
- Whether event-only analysis ships in the first release.
- Ask Ponta pricing: capped, metered or bundled.
- Hosting: the blueprint says Fly.io or Render; Railway was discussed; the frontend could be static on Cloudflare. No decision made. Recommendation given: stay on managed services, keep services as plain containers, EU regions from the start, do not move to AWS now.
- Vite versus Next.js for the frontend (the UI prototype is Vite; the blueprint says Next.js). Leaning Vite static export.
- Final dark palette, away-team colour, heat gradient, filled versus outlined primary buttons, Plotly versus Canvas for the pitch.
- Public naming of Matrix and Plaiq; the IP-protection call on publishing `ANALYTICS_SPECIFICATION.md`.

## Docs that now disagree with the evidence

None known. Reconciled on 15 November 2026: `roster/` placement ([architecture.md](architecture.md), [platform-extraction-task.md](platform-extraction-task.md)), Next.js versus Vite ([architecture.md](architecture.md), [README.md](README.md), [roadmap.md](roadmap.md); noted as undecided, not resolved), home and away tokens ([visual-design.md](visual-design.md), [frontend-design-brief-v1.md](frontend-design-brief-v1.md)), and the decision-quality scale ([ai-layer.md](ai-layer.md)). Still to do in brief v2: the prototype's status colours (values not recorded here), and remaining "Next.js" wording in `architecture.md` and `website.md`, which is covered by the note rather than rewritten.

## Strategy review, 15 November 2026

Read all documents and the live site. Changes made in the blueprint: evidence gates E1 to E3 and a concierge step before the platform ([roadmap.md](roadmap.md)); two wedges ([go-to-market.md](go-to-market.md)); construct validity and coverage ([engine-governance.md](engine-governance.md), [data-tiers.md](data-tiers.md), [platform-extraction-task.md](platform-extraction-task.md)); vendor licence check before the first real export ([legal-compliance.md](legal-compliance.md)); risks 23 to 26 ([risks.md](risks.md)); stale test and function counts corrected ([architecture.md](architecture.md)); brief persona fixed ([frontend.md](frontend.md)); three website flags recorded ([website.md](website.md)). The website repo was not changed.

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

- **CI is observed and green** (run 4, `36ef7d8`). It skips the sample-data tests, so a green CI does not cover the whole suite. Run `PONTA_REQUIRE_SAMPLE_DATA=1 pytest -q` locally before relying on a change. `gh` is not installed; read runs with `curl https://api.github.com/repos/cgvgukapps/ponta-platform/actions/runs`.
- **Install both engine packages in one pip command,** so the local packages resolve each other and pip never looks up an internal name on the public index (distribution names are prefixed `ponta-`).
- **Do not commit `*.egg-info`** (now git-ignored).
- **Sandbox:** writes outside the workspace folder need elevated permissions. Pushes to `main` are held for your approval and need a retry with the approval request. Process listing (`ps`) is blocked.
- **Folder case:** the platform folder is `Ponta-Platform` on disk; macOS ignores case, so `ponta-platform` also works. The venv records the capitalised path.
- **Piping a long job through `grep` hides all output until it finishes,** and the shell reports the pipe's exit status, not the job's.
- **Do not touch the prototype** beyond reading it. It is the validation record.
