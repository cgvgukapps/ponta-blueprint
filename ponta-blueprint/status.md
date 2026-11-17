# Status and handoff

**Last updated:** 17 November 2026. This is the living pick-up note: read it first when resuming, and update it at the end of any session that changes state. It describes the present; the history of how it got here is in git and in each repo's notes. For the plan see [roadmap.md](roadmap.md); for an outside reader's summary see [platform-overview.md](platform-overview.md).

**One-line state:** pre-product, no customers. The engines, the Data check and the semantic builder are built and tested on real open data in `ponta-platform`; the AI layer, API, worker, storage and production web app are not built. A clickable UI prototype on mock data has been through two redesigns and brief v2 items 5.1 to 5.4. A single-file match page built from real engine output is ready for the founder-run concierge step. **Strategy, 17 November 2026:** Ponta is repositioned as a match context model that the AI explains; context packages (possession to episodes) come before the AI build ([context-packages.md](context-packages.md)). Context Stages A and B are built (Gate 1 passed; Gate 2a measured, bars with the founder). Next: the founder's Gate 2a decisions and zone spot check, discovery calls (gate E1) and the concierge step, with the rest of Phase 1 alongside.

## Repositories

| Repo | Local path | Remote (GitHub, `cgvgukapps`) | Position |
|---|---|---|---|
| Blueprint (this repo) | `/Users/cicerovieira/PontaPlan` | `ponta-blueprint` | `main`; tag `blueprint-baseline-2026-11-15` |
| Prototype (frozen) | `/Users/cicerovieira/PONTA-PROTOTYPE` | `ponta-prototype` | `main` at `41bd812`, tag `prototype-validation-final`. Never modify. |
| Platform (production) | `/Users/cicerovieira/ponta-platform` (on disk: `Ponta-Platform`) | `ponta-platform` | `main` at `9017adc` (context Stage B built, 17 November 2026), **not pushed**; `a4f3f19` (Stage A) is the last pushed commit |
| UI prototype | `/Users/cicerovieira/Ponta_Platform_UI` | `ponta-platform-ui` | `main` at `26b7c1d`, pushed; tag `prototype-v0.1` at `4bd8e52` |
| Marketing website | `/Users/cicerovieira/Ponta-Website` | `ponta-website` | `main` at `82a3429`; v2 deployed with `noindex, nofollow` |

## Where things stand

### Engines (`ponta-platform`)

- **Extraction complete** ([platform-extraction-task.md](platform-extraction-task.md), `EXTRACTION_REPORT.md`): `matrix_engine` and `plaiq_engine` as installable packages, all prototype tests carried over, import-graph check, CI.
- **Tests:** 1,069 in total; 1,057 in the full local suite and 12 in the opt-in equivalence test (`PONTA_EQUIVALENCE=1`, about 7 minutes). CI is green but skips data-dependent tests; the local gate is `PONTA_REQUIRE_SAMPLE_DATA=1 pytest -q`.
- **Equivalence:** Plaiq fed through Matrix reproduces the prototype's golden outputs on Metrica Sample Game 3 and SkillCorner 1886347, except 2 of 39,589 decision rows and 12 of 36,792 alternative rows on Metrica (a characterised float near-tie). No band changes. Accepted as the baseline; `engine_version` unchanged.
- **Data check** (`matrix_engine/parity.py`): 12 metrics in 6 families with reasons, measured figures and coverage. On the real SkillCorner match, pass option value and event ΔV are Full and shot quality is Blocked.
- **Semantic builder** (`plaiq_engine/semantic_builder.py`): one pseudonymised row per action, no coordinates; [ai-layer.md](ai-layer.md#the-semantic-football-event) matches the implemented schema.
- **Tools:** `tools/demo.py` (end to end on a sample match) and `tools/match_page_payload.py` (payload for the shareable match page). Golden outputs are regenerable (`tools/golden/README.md`).
- **Open:** the unused `prev_event` parameter of `resolve_identities`. Facade decisions are parked (`docs/MATCH_ANALYSIS_INVENTORY.md`). `apps/api`, `apps/worker` and `apps/web` are empty.

### UI prototype (`ponta-platform-ui`, mock data only)

- Vite, React and TypeScript with a fake in-browser engine, synthetic tracking and hand-written sample brief and answer text. Not connected to any backend.
- **Brief v2:** 5.1 (client-side names), 5.2 (coverage), 5.3 (shareable match page) and 5.4 (review fixes) are done; 5.5 (API mapping) is on hold by founder decision. Ask is also a panel inside each match.
- **Design:** decisions are fixed in [frontend-design-brief-v2.md](frontend-design-brief-v2.md#3-fixed-decisions-do-not-reopen-without-reporting-why) section 3, with the reasoning in the UI repo's `docs/design-notes.md` (sections 6 to 17). Current look: cool near-black palette, serif page headings, white primary buttons, Ponta blue only for focus, selection marks and graphics, grey highlights for selected rows.
- **Shareable match page:** a single offline HTML file from real engine output (SkillCorner 1886347: 1,308 of 1,371 passes and carries scored, 95.4%). Real payloads are generated outside the repos.
- **Open:** Safari print not verified (no page numbers there; `<thead>` repetition unchecked). The UI still shows pressure as a 0 to 1 index with 0.7 as "high"; align it with the engine's 1 to 10 scale once the threshold is chosen.

### Website (`ponta-website`)

- v2 one-product page deployed with `noindex`; hero "Tactical & physical AI that sees the whole pitch, and the athlete inside it."; Sign in disabled.
- Launch waits on gates G1 to G6 ([website-v2-plan.md](website-v2-plan.md#launch-gates-public-announcement-not-the-friends-only-url)): licences (G3), legal pages and sub-processor terms (G4) and the hero's fusion wording (G6) are the substantive ones. Also open: `/security`, `/privacy`, `/terms`, and hero artwork that still shows two engines.

### Blueprint (this repo)

- Reviewed and cleaned on 17 November 2026: status banners on every task spec and plan, stale facts corrected (pressure scale, pitch rendering, product colours, provider list, website state), the old company name "PLAIQ" replaced by Ponta where it meant the business, British spelling throughout, links checked.
- **Context Stage B built (17 November 2026)** in `ponta-platform` ([context-structure-task.md](context-structure-task.md); report `CONTEXT_STRUCTURE_REPORT.md`): P2 team structure (shape, lines, formations, roles), P3 relative zones (lane, third, band against the defending lines, hull, pocket) and P4 phase of play (SkillCorner's vocabulary, Ponta's rules). Roles agree with SkillCorner's by position group for 79% of playing time; phases at Cohen's κ 0.60 (0.58 and 0.61 by half). About 16 s and 4 MB per match for P1 to P4. Gate 2a bars, Stage B thresholds and the 50-card zone spot check are with the founder.
- **Context Stage A built (17 November 2026)** in `ponta-platform` (`plaiq_engine/context/`): the context grid, package registry, Data check entries from packages, golden harness, and P1 possession and game state on all four regression matches; semantic events schema 1.1 adds possession chain, source, score and red-card difference. Report: `CONTEXT_FOUNDATIONS_REPORT.md`.
- **Context foundations spec written (17 November 2026):** [context-foundations-task.md](context-foundations-task.md). It uses a regression set of four local matches and treats SkillCorner's own possession, phase and game-state fields (found in its dynamic events) as comparison data only.
- [platform-overview.md](platform-overview.md) is a dated snapshot for an independent reviewer; also published as a private page for the founder.
- **Context packages adopted (17 November 2026):** [context-packages.md](context-packages.md) records the founder's work plan, adjusted after review, and four decisions: (1) repositioning agreed, the AI build waits for Stage A at the earliest, preferably Stage B; (2) Ponta computes its own context, provider metrics are stored for comparison only; (3) unvalidated fields may reach analysts labelled "estimated, not yet validated", never player-facing views; (4) only Stage A (P0, P1) is committed, Stages B to D planned, P9 in Phase 5. Data-tiers, AI layer, roadmap, risks (27 to 30), go-to-market, competitive landscape and engine governance updated to match.

## Findings that change the plan

1. **`roster/` belongs in `plaiq_engine`, not `matrix_engine`.** Matrix never imports it, and it imports Plaiq's `DEFAULT_PROFILE`. `pitch_config` belongs in `matrix_engine` (both engines import it). Recorded in `ponta-platform/docs/ARCHITECTURE.md`.
2. **`compute_event_pov` row order is non-deterministic across runs; values are deterministic.** The semantic builder sorts explicitly.
3. **The ball is undetected in 32.2% of Metrica Game 3 frames and 26.4% of the SkillCorner match.** The engine returns an empty control grid there; coverage reports it.
4. **Decision quality ranges from 0 to 1.5**, not 0 to 1. Bands: Poor below 0.55, Suboptimal 0.55 to 0.79, Good 0.80 to 0.94, Optimal 0.95 and above.
5. **SkillCorner event ids are inconsistent** (`From` is `'1'`, `To` is `'1.0'`). Normalised in the new code.
6. **Metrica Sample Game 2 is normalised with the home team attacking −x** (the home goalkeeper's mean x is +39.7 m), against `PitchConfig.home_attacks_positive_x=True`. The Metrica CSV loader flips the second half and assumes the first-half direction. Context Stage B reads direction from the data and is unaffected; anything that assumes home attacks +x on Game 2 may be wrong. Recommended: a small Matrix fix with a regression test, as its own task.
7. **Single-source matches have no per-frame confidence** (only `ball_detected` and `players_observed`), which leaves the confidence slot of an AI citation undefined for Tier 1 (see open decisions).
8. **UI prototype:** its fake engine and mock packs must never reach a production bundle, and its hand-written types must become generated types from the API schema.

## Next steps, in order

1. **Track V (validation), now.** Discovery calls with the wedge, intercepted-pass and price questions (gate E1, at least 8 calls); check each prospect's vendor licence before taking a real export ([legal-compliance.md](legal-compliance.md#vendor-licence-check-before-the-first-real-export)).
2. **Context Gate 2a, then Stage C.** The founder agrees the Gate 2a bars and the Stage B thresholds (`CONTEXT_STRUCTURE_REPORT.md`), and does the zone spot check: `python tools/context_spotcheck.py --cache <cache> --out <file outside the repo>` writes 50 cards with tick boxes. Then a Stage C spec (P5 relationships, P6 movement types, P7 space and value). Before or alongside: re-run the role and phase measurements on SkillCorner's open data (10 more matches with phases, reportedly MIT; verify the licence, keep it outside the repos), and the Metrica Game 2 direction fix (finding 6). Analyst agreement (Gate 2b) is still needed before anything is player-facing.
3. **Concierge step** (roadmap Phase 2, step 0): run a prospect's real export locally through the engines and send back the Data check and the match page (gate E2). The page and payload tool exist.
4. **Rest of Phase 1** (not gated by E1), alongside context Stage A: Supabase (Postgres, RLS, Auth), FastAPI skeleton, object storage, the automated cross-tenant access test for Parquet paths, basic ToS ([roadmap.md](roadmap.md#phase-1--platform-foundation-and-engine-extraction)).
5. **Before the AI layer** ([ai-workflows-task.md](ai-workflows-task.md)): context Stages A and B are built; decide the citation confidence for single-source matches and a named high-pressure threshold; choose the LLM provider and put sub-processor terms in place before any real customer data reaches a model.
6. **Frontend, later:** import the UI into `ponta-platform/apps/web` (subtree, or copy and record the source commit); keep the `src/api` signatures and add an HTTP client behind a mock switch; generate types from the OpenAPI schema. The Data check is the first screen that can go real. The API mapping (brief v2, 5.5) waits until then.

**Running in parallel:** the legal gates in [roadmap.md](roadmap.md#track-l--legal-and-compliance-runs-alongside-each-item-gated-to-a-trigger) (basic ToS before the first pilot; LLM sub-processor terms before real customer data reaches a model); the website launch gates above; optionally a `soccertrack-validation-task.md` and the Tier 3 gate status in [data-tiers.md](data-tiers.md) (offered, not requested).

## Open decisions

- **Citation confidence for single-source matches:** what fills the confidence slot (a documented derivation from `ball_detected` and `players_observed`, or an explicit null with the reason). Blocks the AI layer.
- **A named high-pressure threshold** on the engine's 1 to 10 scale, documented and versioned; the UI glossary then follows.
- **Context P1 refinements (open, with recommendations):** a control radius per provider (once a second broadcast match confirms SkillCorner's fit at 2.0 to 2.5 m), a hand-labelled possession sample before Stage B, and movement-based gap inference when a feed needs it ([context-packages.md](context-packages.md#open-decisions)).
- **Context Gate 2a (with the founder):** the proposed bars (roles at least 75% by position group weighted by playing time and 85% with central midfield merged; formations at most 2 changes per team, state and half; phases κ at least 0.55 on each half; zones at least 45 of 50 on the spot check), the Stage B thresholds (three measured: low block 26 m, high block 47 m, break gain 15 m), the Metrica Game 2 direction fix and the SkillCorner open-data download. All in `CONTEXT_STRUCTURE_REPORT.md`, "Decisions for the founder".
- **Context package thresholds:** grid rate, inferred-possession share, possession agreement bar (Gate 1), and the later ones per package; listed in [context-packages.md](context-packages.md#open-decisions), each decided before its package starts, from measurements where possible.
- **Recruit one or two analysts** through discovery calls for context Gates 2 and 3 (formation, phase, movement labels).
- **Evidence-gate thresholds (E1 to E3)** in [roadmap.md](roadmap.md#evidence-gates-proposed-15-november-2026-founder-to-confirm-the-thresholds): confirm or change.
- **Lead wedge** (single-export brief, or fragmentation and provider transition): decided by discovery calls ([go-to-market.md](go-to-market.md#two-wedges-one-to-lead-with-added-15-november-2026)).
- **Provider list on the website:** the page lists Second Spectrum as supported; the engine has it fixture-tested only and the hard rule allows four providers. Change the page or revise the rule with a reason.
- **LLM provider and model** (blocks Phase 2's AI work).
- **Whether event-only analysis ships in the first release.**
- **Ask Ponta pricing:** capped, metered or bundled.
- **Hosting:** Fly.io, Render or Railway for the services, the frontend possibly static on Cloudflare. Recommendation given: managed services, plain containers, EU regions from the start, no AWS now.
- **Vite or Next.js** for the production frontend (leaning Vite static export).
- **Facade decisions** (`MatchAnalysis` input form, rendering helpers, private-name wrappers): parked; recommendations in `ponta-platform/docs/MATCH_ANALYSIS_INVENTORY.md`.
- **Public naming** of Matrix and Plaiq; the IP-protection call on publishing `ANALYTICS_SPECIFICATION.md`.

*Decided and no longer open:* proceeding through the context stages against reference labels, with analyst gates kept for player-facing use (decision 4a, 17 November 2026); the context-packages repositioning and its four decisions, the Gate 1 bar (met) and all P1 thresholds (17 November 2026); the extraction baseline and the SkillCorner Data check (15 November 2026); the UI palette, away colour, heat gradient, primary button style and canvas pitch rendering (fixed in [frontend-design-brief-v2.md](frontend-design-brief-v2.md#3-fixed-decisions-do-not-reopen-without-reporting-why) section 3).

## Docs that now disagree with the evidence

None known after the 17 November 2026 review. Older sections of `architecture.md` and `website.md` still say "Next.js"; each carries a note that the framework is undecided, rather than being rewritten.

## Strategy gaps found but not acted on

The wearable-integration question (OAuth connections, webhooks, PlayerData, Garmin, Apple Watch, Catapult, STATSports Sonra, session-based sync) is **not** in the strategy beyond file upload, Vault-stored credentials and polling. No action requested. If revisited: add OAuth and token-paste connector types, an optional webhook receiver, and a session entity next to matches, all gated behind the Phase 4 health-data legal review. Apple HealthKit is on-device, so it would need a companion app. Vendor access details in the proposal were not verified.

## How to resume

```bash
# Engines
cd /Users/cicerovieira/ponta-platform
source .venv/bin/activate            # recreate if missing, see README
python tools/check_imports.py && ruff check packages tools tests && pytest -q
# Context measurements (write outside the repo; the cache holds pickled sample matches)
python tools/context_golden.py --out /tmp/ponta-context/golden --cache /tmp/ponta-context/cache --check
python tools/context_structure_measure.py --cache /tmp/ponta-context/cache --out /tmp/ponta-context/structure.json

# UI prototype
cd /Users/cicerovieira/Ponta_Platform_UI
npm run dev                          # http://127.0.0.1:5173
npm run typecheck && node scripts/check-i18n.mjs && npm run contrast && npm run build
npm run shots                        # with the dev server running; add "-- --codes" for codes mode
```

Prototype tests use the prototype's own interpreter: `cd /Users/cicerovieira/PONTA-PROTOTYPE && PYTHONDONTWRITEBYTECODE=1 .venv/bin/python -m pytest -q -p no:cacheprovider` (about 2 minutes, expect 964 passed). Regenerate golden outputs: see `tools/golden/README.md` in `ponta-platform`.

## Gotchas

- **CI skips the sample-data tests,** so a green CI does not cover the whole suite. Run `PONTA_REQUIRE_SAMPLE_DATA=1 pytest -q` locally before relying on a change. `gh` is not installed; read runs with `curl https://api.github.com/repos/cgvgukapps/ponta-platform/actions/runs`.
- **Install both engine packages in one pip command,** so the local packages resolve each other and pip never looks up an internal name on the public index (distribution names are prefixed `ponta-`).
- **Do not commit `*.egg-info`** (git-ignored).
- **Context golden summaries are git-ignored** (derived from sample data); only `tests/golden/context/manifest.json` is tracked. After changing a package or threshold version, recapture with `tools/context_golden.py --capture`, or the golden test in `tests/cross_engine/test_context_sample.py` skips.
- **Do not rename or move blueprint documents:** other repos link to them by name (see CLAUDE.md).
- **Folder case:** the platform folder is `Ponta-Platform` on disk; macOS ignores case, so `ponta-platform` also works. The venv records the capitalised path.
- **Piping a long job through `grep` hides all output until it finishes,** and the shell reports the pipe's exit status, not the job's.
- **UI screenshots need the dev server running** and use the system Chrome; real match-page payloads are generated outside the repos and never committed.
- **Do not touch the prototype** beyond reading it. It is the validation record.
