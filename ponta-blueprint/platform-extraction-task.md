# Platform Extraction — Execution Task

> **Status: complete for its scope (15 November 2026).** Both engines run from `ponta-platform` with all prototype tests; the Plaiq-via-Matrix equivalence test, `parity.py`, `semantic_builder.py`, the demo and `EXTRACTION_REPORT.md` are done, and the decisions it raised are recorded in [status.md](status.md#open-decisions). One small item is open: the unused `prev_event` parameter of `resolve_identities`. The rest of Phase 1 (Supabase, FastAPI, storage, the cross-tenant test) was never in this task's scope; see [roadmap.md](roadmap.md#phase-1--platform-foundation-and-engine-extraction).

Self-contained task for a coding agent. It starts from the cleaned `PONTA-PROTOTYPE` (see [prototype-cleanup-task.md](prototype-cleanup-task.md), complete) and creates the production monorepo's engine layer. It does **not** build the API, database, frontend or any LLM feature. Read, in this order, if anything below is ambiguous: [architecture.md](architecture.md) ("Canonical store and AI layer", "Monorepo and repo strategy"), [data-tiers.md](data-tiers.md), [ai-layer.md](ai-layer.md) ("The semantic football event"), and `Matrix.md`.

## Context

`PONTA-PROTOTYPE` (`/Users/cicerovieira/PONTA-PROTOTYPE`) holds two engines that share ingestion code: the Plaiq decision-quality engine (`analytics/`, `statsbomb/`, plus `ingestion/` and `roster/`) and the Matrix engine (`matrix_engine/`, which wraps `ingestion/` one-directionally; it never imports `roster/`, see the import-graph finding in [status.md](status.md)). A 4,600-line Streamlit `app.py` drives Plaiq. The product strategy has changed to one AI-native app: grounded AI reads **semantic football events** produced by the deterministic engines, and every match gets a **Parity Report**. This task extracts the engines into a new monorepo and adds those two missing pieces (`parity.py`, `semantic_builder.py`).

The prototype suite is **964 passing tests** — but 582 of those are one parametrised ring-distance test, so the behavioural core is about 382. "All 964 green" proves the extraction preserved behaviour; it does not prove the new pieces work. They need their own tests.

## Objective

A new repo `/Users/cicerovieira/ponta-platform` (name fixed) containing installable, UI-agnostic `packages/matrix_engine` and `packages/plaiq_engine`, with the prototype's tests carried over and green, a verified Plaiq-via-Matrix equivalence, a working Parity Report, and a semantic-events builder that writes Parquet. `PONTA-PROTOTYPE` is **not modified** except for one tag.

## Constraints

- **Do not change engine math.** Extraction is behaviour-preserving. Any discrepancy found (especially in the equivalence test) is reported, not "fixed" by editing formulas.
- No Streamlit imports anywhere under `packages/`. `app.py` is not ported.
- Dependency direction: `matrix_engine` never imports `plaiq_engine`. `plaiq_engine` may import `matrix_engine`. If the split below produces a cycle, stop and report; do not hide it with local imports.
- No API, Supabase, frontend, worker, queue, or LLM code. No new vendor credentials. No network access except where an existing test already fetches an external file and skips cleanly without it.
- **The unlicensed STATSports sample must not be copied into the new repo** or into any fixture, test artefact or Parquet output. The STATSports provider keeps its current fetch-on-access, anonymise-first, skip-if-unavailable behaviour (`matrix_engine/STATSPORTS_ADDENDUM.md`).
- No player names in any generated artefact (semantic events use pseudonymous refs).
- Match existing code style and docstring conventions.

## Steps

### 0. Preflight

1. In `PONTA-PROTOTYPE`: `pytest -q` and record the exact passed/skipped counts (expect 964 passed). Create a git tag (e.g. `prototype-validation-final`) on the current commit. Make no other change there.
2. Capture **golden outputs from the prototype before any move**, for later equivalence testing: for Metrica sample game(s) and the SkillCorner open-data match, persist (as test fixtures, not committed if large — document how to regenerate) the Plaiq event-level outputs the app computes (`event_pov`, decision quality, movement value/ΔPOV, event ΔV) and a deterministic sample of per-frame pressure/control values.

### 1. Repo skeleton

`ponta-platform/packages/{matrix_engine,plaiq_engine}` each with its own `pyproject.toml`, installable in editable mode from a shared environment; shared `requirements`/lock pinned as the prototype's is; `pytest` runnable from the repo root; `ruff` as in the prototype. Empty `apps/` placeholders are fine. Preserve git history for moved modules where practical (`git filter-repo` or subtree split); if it is costly, say so in the report and skip it.

### 2. Decide and implement the module split

Starting proposal (verify with an import-graph check before moving anything; adjust and justify in the report):

- `matrix_engine/adapters/`: `ingestion/{metrica_io,epts_io,metrica_velocities,skillcorner_io,second_spectrum_io,genius_events_io}.py` and the existing `matrix_engine/providers/*` (which already wrap them), plus `pitch_config.py` (both engines import it). *(Revised: `roster/` was originally listed here. The import-graph check shows Matrix never imports it and it imports Plaiq's `DEFAULT_PROFILE`, so `roster/` moves to `plaiq_engine` instead; `roster.build_roster` lazily imports `replay.load_game`, which is not ported, so it needs a local CSV loader. Recorded in `ponta-platform/docs/ARCHITECTURE.md`.)*
- `plaiq_engine/`: `analytics/`, `statsbomb/`, `roster/`, `ingestion/statsbomb_io.py` (event-only path).
- `matrix_engine/pipeline.py`: a `MatrixPipeline.run(...) -> UnifiedTrackingDataset` façade over the existing `UnifiedTrackingEngine` flow (`providers → sync → identity → sync(refine) → fusion → confidence → quality → export`), behaviour-identical.
- `plaiq_engine/match_analysis.py`: a `MatchAnalysis` façade exposing what `app.py` currently calls ad hoc (inventory those ~30 imported functions first and list them in the report): pressure, control, value surface, `event_pov`, `event_decision_quality`, movement value, player timeline. Wrap, don't rewrite.
- Resolve the open item from the cleanup report: the unused `prev_event` parameter of `resolve_identities` — remove it with its call sites or keep it deliberately, and say which.

### 3. Carry the tests

Move/adapt all prototype tests (imports only). All previously passing tests pass; none are deleted, skipped or weakened. The external-STATSports tests keep skipping cleanly when the source is unreachable. Report starting vs final counts, by package.

### 4. Plaiq-via-Matrix equivalence test

Routing tracking data through Matrix's normalised schema before Plaiq is a real change to numbers' provenance (coordinate normalisation, direction handling, velocity/acceleration derivation). Build a test that runs Plaiq on the sample matches **both** ways — the prototype's direct-ingest path (golden outputs from step 0) and via `MatrixPipeline` — and asserts agreement within an explicit, justified tolerance for event-level outputs. For every difference beyond tolerance: do not alter formulas; characterise it (which stage, which field, magnitude), record it in the report, and mark it as a required `engine_version` change decision for a human. If equivalence cannot be established for a provider, state that provider's Plaiq path stays direct-ingest until resolved.

### 5. `matrix_engine/parity.py` — the Parity Report

Implement `AnalyticsParityReport` per [data-tiers.md](data-tiers.md#analytics-parity-report): per metric family, a state (`Full` / `Degraded` / `Blocked`) and a machine-readable reason, JSON-serialisable (it is stored as JSONB). Inputs derived deterministically from the data and provenance, not from model judgement: provider and mode (full-tracking / event-only / GPS-only), sampling rate, per-frame confidence and dropout/occlusion fractions, which teams are covered, which physiological channels are present, the match's `fusion_tier`. Required rules, each with tests:

- GPS-only (one team): opposition pressure and control zones **Blocked**; physical summaries available.
- Event-only: decision quality **Degraded**, flagged as a separate confidence class.
- Optical at low frame rate or high dropout: affected metrics **Degraded** with the measured figure in the reason.
- No physiological source: any load/fatigue metric **Blocked**.
- Report a **coverage figure** per metric family: the share of events scoreable and the cause of the gap (ball not detected, goalkeeper, set piece), measured, not estimated. Test it against the golden matches (ball undetected in 32.2% of Metrica Game 3 frames and 26.4% of the SkillCorner match).
- Generalise the existing SkillCorner "Analytics Parity" logic in `app.py` rather than re-deriving it; its Full/Degraded/Blocked classification for SkillCorner must be reproduced exactly (test it).

### 6. `plaiq_engine/semantic_builder.py`

Compile a match's results into one row per pass, carry, off-ball run and defensive action, written to `semantic_events.parquet`, matching the schema in [ai-layer.md](ai-layer.md#the-semantic-football-event). Requirements:

- Pure and deterministic: same input, same output (stable ids, ordering).
- **Conditional sections.** `physiological_context` is `null` unless the match has a physiological source *and* the fields exist; never zero-fill. (No heart-rate/IMU ingestion exists yet — this task does not build it; leave the section `null` and tested as such.)
- `player_ref` is a pseudonym (`home_2`); **no names, no raw coordinate arrays** anywhere in the output; zone is a categorical, not x/y.
- `mode` is `full_tracking` or `event_only`; event-only rows are never merged into the full-tracking band.
- Goalkeepers and set-piece events are `not_scored` with a reason.
- Every row carries `schema_version`, `engine_version`, `match_id`, `timestamp_s`, and the frame-level confidence/parity annotation.
- Also write `trajectories.parquet` from the unified dataset (schema in [architecture.md](architecture.md#storage-hybrid-by-access-pattern)), with physiological columns present only if a source exists.

Tests: schema validation; determinism; **no-names and no-coordinate-arrays assertions**; a spot-check that sampled rows' DQ/POV/pressure values equal direct calls into the engine for the same events; conditional-section behaviour for full-tracking vs event-only vs GPS-only fixtures; GK/set-piece `not_scored`.

### 7. Demo

A script (not an app) that takes one sample match through `MatrixPipeline` → `parity` → `MatchAnalysis` → `semantic_builder` and prints the Parity Report and the first rows of both Parquet files, so the result is inspectable without any UI.

## Acceptance criteria

- `PONTA-PROTOTYPE` unchanged except the tag; its suite still reports the original counts.
- `ponta-platform` installs; both packages import with no Streamlit and no dependency cycle (import-graph check included in CI).
- All carried tests pass; counts reported by package; zero deleted/weakened tests.
- Equivalence test exists, passes within stated tolerance or its failures are fully characterised in the report.
- `parity.py` and `semantic_builder.py` exist with the tests above; Parity rules reproduce the prototype's SkillCorner classification exactly.
- No unlicensed STATSports data, no names, no raw coordinate arrays in any repo artefact or output.
- Demo runs end-to-end on at least one Metrica and one SkillCorner sample match.

## Explicitly not in scope

FastAPI, Supabase, auth, worker/queue, object-storage code, DuckDB query tooling, any LLM or AI workflow ([ai-workflows-task.md](ai-workflows-task.md)), the web frontend, heart-rate/IMU ingestion, Tier 2/3 productisation, and any change to engine formulas.

## Final report (`EXTRACTION_REPORT.md`, repo root)

Short and honest, in the style of `CLEANUP_REPORT.md`: the final module split and why (with the import graph); test counts before/after by package; the equivalence results including every difference and its magnitude; the Parity rules implemented and any case where the data did not support a clean classification; semantic-builder decisions; the `prev_event` resolution; what history was or wasn't preserved; and any decision left for a human.
