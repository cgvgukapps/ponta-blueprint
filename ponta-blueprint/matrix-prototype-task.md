# Matrix Prototype — Execution Task

This is a self-contained task description for a coding agent building inside `/Users/cicerovieira/PONTA-PROTOTYPE`. It doesn't assume the agent has any other context — everything needed is below or in the files referenced.

## Context

Ponta is building two products: **Matrix** (a vendor-agnostic tracking-data reconciliation engine — the priority) and **Plaiq** (an existing off-ball analytics engine). Full product context is in `/Users/cicerovieira/PontaPlan/ponta-blueprint/Matrix.md` — read that first if anything below is ambiguous. This task is the first prototype build for Matrix, done as a validation exercise, not a production build.

The target repo, `PONTA-PROTOTYPE` (named `PLAIQ-PROTOTYPE` at the time this task was originally run — see `Matrix.md` for the rename), already contains a well-tested analytics/ingestion codebase (159 passing tests) for **Plaiq**: `analytics/`, `ingestion/`, `roster/`, `statsbomb/`, `pitch_config.py`, driven today by a Streamlit app (`app.py`). This task does not touch any of that except to import from `ingestion/`, `roster/`, and `statsbomb/` — treat those as a stable, tested dependency, not something to modify.

## Objective

Build a new `matrix_engine/` package inside `PONTA-PROTOTYPE` that proves Matrix's core reconciliation capabilities using **only data already present in the repo** — no new vendor credentials, no network calls, no new external data. Prove it with automated tests plus a runnable demo script that reports quantitative validation metrics.

## Available data (already in the repo — do not fetch anything else)

- `sample-data/data/Sample_Game_1/`, `Sample_Game_2/`, `Sample_Game_3/` — Metrica sample tracking (25 Hz, home/away CSVs or EPTS-style files) + events. This is your **ground-truth source** for Track B.
- `sample-data/data/skillcorner/matches/1886347/` — real SkillCorner open-data match: `1886347_match.json`, `1886347_tracking_extrapolated.jsonl`, `1886347_dynamic_events.csv`. This is your **second real vendor** for Track A.
- Existing adapters to reuse, not rewrite: `ingestion/metrica_io.py`, `ingestion/epts_io.py`, `ingestion/metrica_velocities.py`, `ingestion/skillcorner_io.py`, `roster/build_roster.py`, `statsbomb/identity.py`, `pitch_config.py`.
- `requirements.txt` already includes `numpy`, `pandas`, `scipy` — `scipy` is available for interpolation/filtering if needed (e.g. `scipy.interpolate`, `scipy.signal`). Don't add new dependencies unless genuinely necessary; if you do, add them to `requirements.txt` and justify why in your final report.

There is **no real wearable/GPS data and no two real recordings of the same match from different vendors** anywhere in this repo. Do not simulate this by pretending two unrelated matches are the same fixture — keep Track A (real, different matches, different vendors) and Track B (one real match, one synthetic degraded copy) clearly separate, as described below.

## Deliverables

### 1. Package skeleton

```
matrix_engine/
  __init__.py            # exposes a top-level UnifiedTrackingEngine facade class
  providers/
    __init__.py
    metrica.py            # wraps ingestion/metrica_io.py + metrica_velocities.py, outputs the common schema below
    skillcorner.py         # wraps ingestion/skillcorner_io.py, outputs the common schema below
  schema.py               # defines the common normalized tracking schema (see below) + a validator
  sync.py                 # timestamp synchronisation / resampling to a common clock
  identity.py             # cross-source player ID matching
  fusion.py               # multi-source trajectory fusion
  confidence.py           # per-frame confidence scoring
  quality.py              # drift detection + data-quality gating
  export.py               # writes the standardized output (CSV/parquet) from a fused result
  testing/
    __init__.py
    synthetic_source.py    # generates a degraded synthetic "second source" from a ground-truth DataFrame
  README.md               # see "Final report" section below
```

No Streamlit imports anywhere in `matrix_engine/` — it must be usable from a plain Python script or pytest, same constraint as the existing engine code.

### 2. Common normalized schema (`schema.py`)

Define one pandas `DataFrame` schema every provider adapter must output, at minimum:

| column | type | notes |
|---|---|---|
| `timestamp` | float (seconds, match-relative) | zeroed at kickoff or first frame, per source |
| `frame` | int | source-native frame index, kept for traceability |
| `player_id` | str | normalized player identifier (use existing roster/identity resolution where applicable) |
| `team` | str | `home` / `away` |
| `x`, `y` | float | metres, using the pitch convention already established in `pitch_config.py` |
| `source` | str | e.g. `metrica`, `skillcorner`, `synthetic_gps` |
| `sample_rate_hz` | float | native sample rate of this source |

Add a `validate_schema(df)` function that checks required columns/dtypes and raises a clear error if violated — every provider adapter and the fusion output should be validated against this before being trusted downstream.

### 3. Track A — cross-provider normalisation (real data, different matches)

- `providers/metrica.py`: load one Metrica sample game via `ingestion/metrica_io.py`/`epts_io.py`, output the common schema.
- `providers/skillcorner.py`: load the SkillCorner match at `sample-data/data/skillcorner/matches/1886347/` via `ingestion/skillcorner_io.py`, output the common schema.
- A test proving both outputs pass `validate_schema()`, have consistent units/coordinate ranges (e.g. both within pitch bounds from `pitch_config.py`, not one in metres and one in some other unit), and are structurally comparable (same columns/dtypes) despite coming from genuinely different vendor formats.

This track does **not** attempt fusion — Metrica's Sample_Game and the SkillCorner 1886347 match are different fixtures. It only proves normalisation works across real, structurally different vendors.

### 4. Track B — synthetic dual-source fusion (one real match, one synthetic degraded copy)

**`testing/synthetic_source.py`** — given a ground-truth normalized DataFrame (from `providers/metrica.py` on one Sample_Game), produce a degraded copy simulating an independent second source, with independently configurable:

- **Downsampling** — e.g. resample from 25 Hz to 10 Hz.
- **Position noise** — additive Gaussian noise on `x`/`y` (configurable std dev, e.g. 0.3–1.0 m to emulate GPS-grade rather than optical-grade accuracy).
- **Frame dropout** — randomly drop a configurable fraction of frames (simulate GPS signal loss/occlusion).
- **Identity shuffling** — remap `player_id` values to a different, shuffled label space per team (simulate the two sources not sharing a common ID scheme), while returning the ground-truth mapping separately for the test to check against later (the pipeline itself must not see this mapping).
- **Timestamp drift** — add a constant offset and/or slow linear drift to `timestamp`.

Return the degraded DataFrame (`source = "synthetic_gps"`) plus the ground-truth mapping/params used, for test assertions only — the reconciliation pipeline itself only ever sees the degraded data plus the original ground-truth source, exactly as if they were two independent real feeds.

**`sync.py`** — align the ground-truth source and the synthetic degraded source onto a common time base (resample/interpolate as needed given their different rates and drifted clocks).

**`identity.py`** — given two time-synced sources with independent ID label spaces, match players across sources (e.g. nearest-trajectory / trajectory-correlation matching over a window, not just nearest-single-frame) and recover a mapping between them.

**`fusion.py`** — given the two synced, identity-matched sources, produce one fused trajectory per player per frame (a confidence-weighted combination is sufficient for a prototype — e.g. weight inversely by each source's estimated noise, or a simple complementary/Kalman-style filter if straightforward; don't over-engineer this for a first prototype).

**`confidence.py`** — emit a per-frame, per-player confidence score reflecting agreement between the two sources and data availability (e.g. lower confidence during dropout gaps or high inter-source disagreement).

**`quality.py`** — detect the injected timestamp drift (flag it) and detect dropout gaps (flag them), on both a drifted and a clean (no-drift) synthetic run, without false-positiving on the clean run.

**`export.py`** — write the fused result (fused `x`, `y`, derived velocity/acceleration, `confidence`, `source_attribution` per frame per player) to CSV, matching the output columns described in `Matrix.md`'s "Outputs" section.

### 5. Tests (`tests/matrix_engine/`)

Follow existing repo conventions (pytest, fixtures similar in spirit to `tests/skillcorner_fixture.py`). At minimum:

- `test_schema.py` — schema validation passes/fails correctly on good/bad DataFrames.
- `test_providers.py` — Track A: both real providers produce valid, comparable schema output.
- `test_synthetic_source.py` — the degradation generator actually produces the requested noise/dropout/drift/shuffle (sanity-check the generator itself before trusting it as a test fixture).
- `test_sync.py` — two sources with different rates/drift end up on a common, correctly-aligned time base.
- `test_identity.py` — shuffled IDs are correctly recovered above a stated accuracy bar; report the actual measured accuracy in the final report even if it's not 100%.
- `test_fusion.py` — **the core claim to prove**: fused-output position RMSE vs. true ground truth is measurably lower than the degraded-source-alone RMSE vs. ground truth. Report both numbers.
- `test_confidence.py` — confidence scores are measurably lower in injected-dropout/high-noise regions than in clean regions.
- `test_quality.py` — drift is correctly flagged on the drifted run and not on the clean run.

Run the **existing** full test suite (`pytest`) before starting and again at the end — confirm zero regressions to the 159 existing tests. Do not modify existing test files or source files outside `matrix_engine/`.

### 6. Demo script

`scripts/matrix_demo.py` — runs the full pipeline end-to-end on one Sample_Game:

1. Load ground truth via `providers/metrica.py`.
2. Generate a degraded synthetic second source via `testing/synthetic_source.py`.
3. Run `sync` → `identity` → `fusion` → `confidence` → `quality` → `export`.
4. Print a summary: RMSE (degraded-alone vs. fused), identity-matching accuracy, average confidence, drift-detection result.
5. Save a simple matplotlib plot (to a file, not interactive) overlaying ground-truth vs. degraded vs. fused trajectory for 1-2 players over a short window, for visual sanity-checking.

### 7. Final report (`matrix_engine/README.md`)

Short and honest, matching this blueprint's existing tone (see `risks.md` for the style to match):

- What was proven (with actual numbers: RMSE reduction, identity-matching accuracy, confidence/drift correctness) — not just "it works."
- What wasn't proven (real multi-vendor overlap on the same match, real wearable data, production-scale performance, anything requiring real vendor credentials).
- What real data/access would be needed to validate further.
- Any new dependency added and why.

## Constraints

- Do not modify `analytics/`, `ingestion/`, `roster/`, `statsbomb/`, `app.py`, or existing tests — only import from them.
- No network calls, no new vendor credentials, no data beyond what's already in `sample-data/`.
- Keep new dependencies minimal; justify any addition to `requirements.txt` in the final report.
- Match existing code style/conventions (docstrings, structure of existing `ingestion/*_io.py` modules as a reference).

## Acceptance criteria

- `pytest` passes in full (existing 159 tests + all new `matrix_engine` tests), with zero modifications to existing test files.
- `scripts/matrix_demo.py` runs end-to-end offline and prints a metrics summary showing fused RMSE measurably below degraded-source-alone RMSE.
- Identity re-matching accuracy is measured and reported (not assumed) on the synthetic shuffle test.
- Drift detection correctly distinguishes a drifted run from a clean run.
- `matrix_engine/README.md` gives an honest account per the "Final report" section above.

## Explicitly not in scope

Real integrations with TRACAB, Hawk-Eye, STATSports, Catapult, KINEXON, Polar, Opta, or Wyscout; any production repo split, API/service layer, or deployment; any UI; legal/ToS/GDPR review; performance/scale testing.
