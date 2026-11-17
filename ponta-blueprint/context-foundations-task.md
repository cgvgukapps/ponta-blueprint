# Context Foundations (P0 and P1) — Execution Task

> **Status: complete (17 November 2026).** Gate 1 passed against the agreed bar; thresholds agreed. Executed in `ponta-platform`; results in its `CONTEXT_FOUNDATIONS_REPORT.md`. P0 and P1 run on all four regression matches (about 2.5 s and 0.8 MB each). Proximity possession agrees with event-based possession on 93.6% to 95.7% of compared time on the Metrica matches. Waiting on the founder: the Gate 1 bar, agreeing the measured thresholds, and one control radius or one per provider. Kept as the record of what was asked.

Self-contained task for a coding agent working in the `ponta-platform` monorepo (`/Users/cicerovieira/Ponta-Platform`). Read [context-packages.md](context-packages.md) first: it is the specification (principles, package contract, storage convention); this document is the work breakdown. Also relevant: [ai-layer.md](ai-layer.md#the-semantic-football-event) (the semantic event schema), [data-tiers.md](data-tiers.md#context-packages-by-data-type) (what each kind of data supports), and `EXTRACTION_REPORT.md` and `docs/ARCHITECTURE.md` in the monorepo.

## Context

**Ponta** compiles a club's match data into one quality-scored dataset and lets a grounded AI explain the match, citing every claim. Today the AI's evidence stops at single actions (one semantic event per pass, carry, run or defensive action). Context packages add descriptions of the whole match. This task builds the foundation every package uses (P0) and the first package, possession and game state (P1), and **measures** the data so the first thresholds can be set from numbers rather than guessed.

**What exists in `ponta-platform` (main at `930d652` or later):**

- `matrix_engine`: provider ingestion and normalisation. `matrix_engine/match.py` defines `NormalisedMatch` (fields `provider`, `match_id`, `fps`, `pitch`, `frames` with `frame, period, time_s`, `tracking` in the Matrix long schema, `ball` with `frame, x, y` only where detected, `events` in the vendor's columns, `slots`) and the loaders `load_metrica_csv_match(data_dir, game_id)` (games 1 and 2), `load_metrica_epts_match(data_dir, 3)` and `load_skillcorner_match(match_dir, match_id)`. Coordinates are metres, centred at (0, 0), home attacking +x for the whole match. `matrix_engine/parity.py` builds the Data check (`ParityInputs`, `MetricParity`, `AnalyticsParityReport`, `classify`, `parity_for_match`).
- `plaiq_engine`: `MatchAnalysis` (`plaiq_engine/match_analysis.py`) wraps the engine for one `NormalisedMatch`. `MatchAnalysis.possession` is **event-anchored** possession from `infer_possession` (`analytics/hex_analytics.py`): gain events (pass, shot, set piece, recovery) set a team from their start frame, loss events clear it, the state is carried forward. `MatchAnalysis.frame_analytics` holds pressure per player for **every frame**. `statsbomb/stitch.py` has `get_possession_chains` for event-only data. `semantic_builder.py` writes `semantic_events.parquet` and `trajectories.parquet`.
- Import rule, enforced by `tools/check_imports.py`: `matrix_engine` never imports `plaiq_engine`.
- Sample data is read through `matrix_engine/sample_data.py` (`PONTA_SAMPLE_DATA`, default `../PONTA-PROTOTYPE/sample-data`). It is licensed or unverified and **never** enters the repo. Tests that need it skip without it; `PONTA_REQUIRE_SAMPLE_DATA=1 pytest -q` is the full local gate.

**The regression set (four tracking matches, all local):**

| Match | Loader | Events | Notes |
|---|---|---|---|
| Metrica Sample Game 1 | `load_metrica_csv_match(…, 1)` | CSV: PASS, RECOVERY, BALL LOST, CHALLENGE, SET PIECE, BALL OUT, SHOT, FAULT RECEIVED, CARD | Known issue: `to_single_playing_direction` may not flip second-half **event coordinates** (see `EXTRACTION_REPORT.md`). Use event frames and teams here, not event coordinates. |
| Metrica Sample Game 2 | `load_metrica_csv_match(…, 2)` | Same | Same issue |
| Metrica Sample Game 3 | `load_metrica_epts_match(…, 3)` | EPTS JSON | Loading takes about 100 s; cache within a run |
| SkillCorner 1886347 | `load_skillcorner_match(…, 1886347)` | Dynamic events CSV | 10 Hz broadcast tracking; players off screen are extrapolated |

**Game-state sources in the data:**

- **Metrica:** goals appear as `SHOT` events whose subtype ends in `-GOAL` (for example `ON TARGET-GOAL`, `HEAD-ON TARGET-GOAL`). One subtype is ambiguous, `WOODWORK-GOAL`: do not guess; report how it is treated and why. `GOAL KICK` subtypes are not goals. Cards are `CARD` events (only `YELLOW` appear in games 1 and 2; check game 3).
- **SkillCorner:** `1886347_match.json` has the final score (`home_team_score`, `away_team_score`), period frame ranges, and per-player `goal`, `own_goal`, `yellow_card`, `red_card` counts (not times).
- **SkillCorner's dynamic events also carry the provider's own context:** player possessions, `team_in_possession_phase_type`, `game_state`, `team_score`. Under founder decision 2 these are **comparison data only**, never Ponta's source. The file also contains **real player names**: they must never appear in any output, log, test fixture or report.

## Preconditions (stop and report if not true)

- `ponta-platform` is at `930d652` or later and the full local gate passes.
- The four regression matches load locally.
- [context-packages.md](context-packages.md) has been read.

## Objective

1. **P0:** a context-package framework that any later package plugs into: a fixed time grid, a frame-level context table, a package registry, Data check entries from packages, and a golden-output harness.
2. **P1:** possession and game state on every grid frame of every regression match, each value tagged observed or inferred, with possession chains and Data check entries.
3. **Measurements** that let the founder agree the first thresholds: ball-detected share, inferred-possession share, and agreement with event-based possession.

## Hard constraints

1. **Do not change existing engine formulas or existing semantic event fields.** Everything here is additive. New semantic event columns are allowed; bump `schema_version` and say so.
2. **Respect the import rule.** `matrix_engine` must not import `plaiq_engine`. Context packages that need Plaiq live on the Plaiq side; anything `matrix_engine` must know (for example the Data check entry type) is defined in `matrix_engine` and filled in by the packages.
3. **No coordinates for the AI.** Frame-level context may contain metres (it feeds later packages and the replay), but anything added to `semantic_events` is labels and numbers about the action, never position arrays.
4. **No names, no licensed data in the repo.** Players are pseudonymous refs (`home_10`). Outputs, golden data and caches go outside the repo or into git-ignored paths, like the existing golden outputs.
5. **Deterministic.** The same inputs produce byte-identical outputs (explicit sorts, no time-dependent values in compared files).
6. **Every field is defined.** Each output field has a written definition, unit, observed or estimated tag, and the package version that produced it.
7. **Thresholds live in one versioned config,** each with a status: `proposed` (chosen by you, with the reason), `measured` (from this task's numbers) or `agreed` (set by the founder; you cannot set this). Nothing threshold-like is hard-coded elsewhere.
8. **Measure before proposing.** Every proposed threshold cites the measurement it came from.
9. **Tests skip without sample data,** using the existing fixture pattern, and run in the full local gate.

## Steps

### 0. Preflight

Run the full local gate and record the counts. Load all four matches; record frames, native rate, period ranges, ball-detected share, and load time. Stop and report if a match fails to load.

### 1. P0 — foundations

1. **Placement.** Decide where context packages live (recommended: a `plaiq_engine/context/` subpackage, since later packages need Plaiq's pressure and pass logic; a separate package is acceptable if you justify it). Record the decision in `docs/ARCHITECTURE.md` and keep the import check passing.
2. **Context grid.** Resample each match onto a fixed grid (5 Hz proposed, configurable), keyed by `match_id` and `frame_ts` (seconds on the match clock) with `period`. Map each grid point to the nearest native frame within half a native period; keep that native frame number for refinement. **Never invent data:** where the ball was not detected at the mapped frame, it stays undetected on the grid; where a player was not observed, they stay unobserved.
3. **`context_frames.parquet`.** Base columns `match_id`, `frame_ts`, `period`, `native_frame`, then one block of columns per package, prefixed by package name (for example `p1_possession`), plus one `<package>_version` column per package. Written beside the existing artefacts in a temporary output directory.
4. **Package registry.** Each package declares name, version, dependencies, the fields it produces (definition, unit, observed or estimated tag) and its Data check rules. The registry resolves run order from dependencies, refuses cycles and missing dependencies, and runs packages on one match.
5. **Data check entries from packages.** Extend the Data check so packages can add their own family and metrics (state, reason, measured figure, coverage line) without `matrix_engine` importing package code. Existing families and outputs must not change.
6. **Golden harness.** Run all registered packages on the regression set and compare against stored golden outputs (git-ignored, regenerable, like `tools/golden/`). Report differences by package and field.
7. **Empty test package.** A package with one trivial field proves the path end to end: grid, registry, frame file, Data check entry and golden comparison.

### 2. P1 — measure first

Before fixing any rule, measure and report, per match and per period:

- the ball-detected share on the grid, and the distribution of detection-gap lengths;
- for a sweep of candidate control radii and ball-speed limits, the share of detected-ball frames where one team's player is clearly closest (in control), both teams are close (contested), or neither is;
- agreement between proximity possession and the event-anchored possession (`MatchAnalysis.possession`) on the three Metrica matches: share of frames agreeing, a confusion matrix, and the longest disagreeing stretches with their timestamps;
- for SkillCorner, agreement with the provider's player-possession events, reported as a comparison only.

### 3. P1 — build

1. **States:** `home`, `away`, `contested`, `dead_ball`, plus `unknown` where nothing supports a label.
2. **Observed possession** from ball proximity and control when the ball is detected, using the proposed thresholds from step 2.
3. **Inferred possession** during detection gaps, from player movement and the last observed state, tagged `inferred`; the event-anchored possession is the fallback and is tagged with its own source. Every frame records its source: `observed`, `inferred` or `events`.
4. **Dead ball** from events where available (ball out, set piece until the restart is played); otherwise `unknown`, never guessed.
5. **Possession chains:** id, team, start and end time, how each began and ended (from events where available, otherwise from label transitions). Report the share of chains whose start and end type is known.
6. **Game state on every grid frame:** score for each team, minute, red cards per team, dead ball, set-piece type. Derive from events (Metrica) or `match.json` (SkillCorner: final score is known, goal times may not be; say which fields are therefore unknown). Add an **upload input** for matches without event data: a small, validated JSON (final score, goal times, red cards with times) that the package reads when present.
7. **Data check entries:** possession `Full` or `Degraded` by the inferred share against a threshold proposed from step 2 (status `proposed` until the founder agrees it); game state `Blocked` when unknown, `Degraded` when only partly known (for example a final score without goal times). Each entry carries a coverage line.
8. **Semantic events:** add `possession_chain_id`, `possession_source`, and game state at the action (score difference for the actor's team, minute, red-card difference) as new nullable columns; bump `schema_version`. Existing columns and values must not change.

### 4. Documentation

- A definition spec for every P0 and P1 field, in a `docs/context/` vocabulary file in the monorepo: meaning, formula, unit, observed or estimated, thresholds by name, known limits.
- The thresholds config, with each value's status, reason and source measurement.

## Acceptance criteria

- The empty test package runs end to end on all four regression matches and writes a stamped frame file and a Data check entry.
- Every grid frame of every regression match carries a possession label and its source (`observed`, `inferred`, `events`, or `unknown` with a reason), and game state with any unknown parts marked.
- The step 2 measurements exist for every match, and every proposed threshold cites them.
- Agreement with event-anchored possession is reported for the three Metrica matches. **Gate 1 is not declared passed here:** the founder sets the bar from these numbers.
- Outputs are byte-identical across two runs.
- The full local gate passes; the import check passes; CI stays green with data tests skipped; no existing test is weakened or removed; existing semantic event values and the existing Data check families are unchanged (the equivalence and golden tests still pass).
- No player name, licensed data or coordinate array reaches the repo or any AI-facing field (asserted by tests).
- Processing time and storage added per match are measured and reported.

## Explicitly not in scope

P2 and later packages; anything in the AI layer, API, worker or UI; ingesting the provider's possession or phase fields as Ponta outputs (comparison only); fixing the Metrica second-half event-coordinate issue (report whether it affects anything here); any change to decision quality or other existing formulas; choosing the agreement bar or agreeing thresholds.

## Final report (`CONTEXT_FOUNDATIONS_REPORT.md`, repo root)

Short and honest:

- What was built and where (placement decision, schemas, registry, Data check extension).
- The measurements table per match: ball-detected share, gap lengths, control shares across the sweep, inferred share, agreement with event-anchored possession (and the SkillCorner comparison), chain coverage.
- Each proposed threshold with its value, reason and source measurement, ready for the founder to agree or change.
- Game-state derivation per match, including how `WOODWORK-GOAL` was treated, and which fields are unknown where.
- Runtime and storage added per match.
- Anything that differs from [context-packages.md](context-packages.md), and every decision left for a human.
