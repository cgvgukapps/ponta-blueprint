# Matrix — Real STATSports Provider Adapter (Follow-up Task)

Self-contained execution task for a coding agent building inside `/Users/cicerovieira/PONTA-PROTOTYPE`. Follows on from [matrix-prototype-task.md](matrix-prototype-task.md), whose `matrix_engine` package already exists and passed validation (see `matrix_engine/README.md` in that repo for the numbers: 320 tests passing, 24.5% fusion RMSE improvement on synthetic dual-source fusion, 100% identity-matching on the synthetic shuffle test, etc.). This task **extends** that package — it does not create a new one.

## Why this task exists

The Matrix prototype's biggest honestly-flagged gap was: *"No wearable or GPS data exists in this repo. `synthetic_gps` is a label on downsampled, noised optical data."* A web/GitHub search turned up a real, raw STATSports Apex GPS+IMU export (`bakero08/Statsports-data-analytics-UCD` on GitHub) from real League of Ireland matches. It does **not** solve the bigger blocker (no optical tracking exists for the same matches, so real fusion still can't be validated against ground truth) — but it does let Matrix ingest and normalise a genuine wearable vendor's raw export for the first time, instead of only a synthetic stand-in. That's a real, scoped improvement, not a full fix, and the report this task produces must say so plainly.

## Data source and provenance — read this before touching any data

- **Source**: `maindata.csv` from [github.com/bakero08/Statsports-data-analytics-UCD](https://github.com/bakero08/Statsports-data-analytics-UCD), accessed directly from its raw URL (`https://raw.githubusercontent.com/bakero08/Statsports-data-analytics-UCD/main/maindata.csv`, ~51 MB) — fetched on access, not pre-downloaded and kept. Real per-sample STATSports Apex export: `Player Display Name, Time, Lat, Lon, Speed (m/s), Heart Rate (bpm), Hacc, Hdop, Quality of Signal, No. of Satellites, Instantaneous Acceleration Impulse, Accl X, Accl Y, Accl Z, Gyro X, Gyro Y, Gyro Z`, real UCD AFC (League of Ireland) matches from the 2022 season, one contiguous block of rows per named player.
- **This is not a licensed open-data program** the way Metrica's and SkillCorner's samples are. It's real, name-attributed player data (including heart rate — physiological data) posted to a public student capstone-project GitHub repo, with no stated license and no visible evidence STATSports, UCD AFC, or the players consented to third-party reuse. Treat it as sensitive, not as a casual sample dataset.

**Hard rules for handling this data, non-negotiable for this task — fetch-on-access, not a permanent local copy:**

1. **The adapter reads from the source URL directly by default.** No manual "download this file first" step should be required to use it — a local path is a convenience override (e.g. for a developer who already has a copy open), not a prerequisite. A persistent, durably-stored local copy is exactly the wrong shape of caution for data this sensitive: it creates a standing artifact that outlives the validation task and is one accidental `git add .` away from a much worse problem than a transient, in-memory read would ever be.
2. **If disk caching is used at all** (reasonable given the file is ~51 MB and re-fetching on every `pytest` invocation during iterative development is wasteful), it must go through `tempfile` or an explicitly gitignored, clearly-disposable cache location — never a directory or naming convention implying permanence, and never inside the repo's tracked tree. Document in the loader's docstring that this cache is safe to delete at any time, and **delete it once this task's validation is complete** — the point of fetching on access is to minimize how long a copy of this data exists anywhere, not just to keep it out of git.
3. **Anonymize at load time, by default, with no opt-out in any code path that writes output.** `Player Display Name` must never survive past the provider adapter's first transformation step — replace it with an opaque, stable identifier (e.g. a hash of the name, truncated, or a sequential `player_001` scheme) before the data is returned to any caller. No real name may appear in a DataFrame, a CSV written to disk, a test assertion string, a print statement, or the final report.
4. **Tests that need this data must skip cleanly, not fail, whenever it isn't reachable** — which now covers both "no local override configured" *and* "network unavailable, source URL unreachable, or the repo has been taken down." This matters more here than it would for a pre-downloaded fixture: relying on a live URL into someone else's unlicensed, sensitive-data repo is inherently fragile — it can disappear at any time, for good reason, and the test suite must degrade gracefully when that happens rather than assuming today's availability is permanent. `pytest` must stay fully green for anyone — including a future CI run, or a run six months from now after the source repo no longer exists — exactly as the original task required for the existing 320 tests.
5. Do not add this dataset, its URL, or its cache path as a default/required fixture anywhere. It is opt-in, ephemeral validation scaffolding — same spirit as `matrix_engine/testing/synthetic_source.py`, but for a real external source instead of a generated one.

If any of this is ambiguous when you're actually building, err toward more caution (less retained, more disposable), not less.

## Objective

Build `matrix_engine/providers/statsports.py`, extending the existing `matrix_engine` package, that parses a real STATSports raw export into the common Matrix schema (`matrix_engine/schema.py`), with a defensible (if approximate) lat/lon-to-pitch coordinate transform, real native-rate detection, and the vendor's own per-sample quality signals carried through as an optional enrichment — then wire that quality signal into `confidence.py`/`quality.py` so it's actually used, not just parsed and ignored.

## Deliverables

### 1. `matrix_engine/providers/statsports.py`

```python
DEFAULT_STATSPORTS_SOURCE = (
    "https://raw.githubusercontent.com/bakero08/Statsports-data-analytics-UCD/main/maindata.csv"
)

def load_statsports_source(
    source: str = DEFAULT_STATSPORTS_SOURCE,  # URL by default; a local path also works
    *,
    player_names: list[str] | None = None,    # subset to load, or None for all
    anonymize: bool = True,                    # no code path should ever set this False
    max_rows: int | None = None,
    cache_dir: str | None = None,               # optional disposable cache for repeated test runs;
                                                 # never permanent, never inside the repo tree
) -> pd.DataFrame:
    ...
```

`MATRIX_STATSPORTS_SAMPLE_PATH` (a local file) and `MATRIX_STATSPORTS_SAMPLE_URL` (an alternate URL) are both optional env-var overrides for `source`; absent either, the function fetches `DEFAULT_STATSPORTS_SOURCE` directly. A network failure, timeout, or 404 (source taken down) must be caught and treated the same as "not configured" by callers — see the test-skipping rule above.

- Parse the raw CSV schema above into `matrix_engine.schema`'s common columns (`timestamp`, `frame`, `player_id`, `team`, `x`, `y`, `source="statsports"`, `sample_rate_hz`), plus a new optional `native_quality` column (see schema extension below).
- **Anonymization**: hash or sequentially remap `Player Display Name` to `player_id` before anything else touches the data. Verify with a dedicated test that the literal name strings from the source file never appear anywhere in the returned DataFrame's values.
- **Team**: this export has no team column (it's one club's own training/match export) — assign everything to a single synthetic team label (e.g. `"statsports"`) rather than guessing `home`/`away`; document this limitation rather than fabricating a split.
- **Coordinate transform (lat/lon → pitch metres)**: there is no ground-truth pitch survey (corner coordinates, orientation) available for this data, so implement a clearly-labelled *estimate*, not a calibrated transform:
  - Project lat/lon to a local planar frame using an equirectangular or azimuthal-equidistant approximation centred on the session's own GPS centroid (fine at pitch scale — under ~150 m, projection distortion is negligible).
  - Estimate pitch orientation via PCA on the projected point cloud (the long axis of player movement over a full match approximates the pitch's long axis) and rotate so that axis aligns with the schema's x-axis convention.
  - Log/report the recovered span in each axis as a sanity check against real-world football pitch dimensions (roughly 90–120 m × 45–90 m) — this is a plausibility check, not a validation, and the report must say so.
- **Native sample rate**: measure from consecutive timestamp deltas per player, don't assume a fixed value.
- **`native_quality`**: derive a 0–1 (or documented raw-passthrough) score from `Quality of Signal`, `Hdop`, and `No. of Satellites` — simple and explainable is fine for a first pass (e.g. a weighted combination or a lookup on `Quality of Signal`'s own scale, whatever the data actually contains — inspect real values before deciding, don't guess the scale).

### 2. `schema.py` extension

Add `native_quality` as an **optional** column: extend `validate_schema()` to accept it when present and continue to pass DataFrames without it (Metrica/SkillCorner providers and all of Track A/B must be unaffected — add a regression test confirming this). Document in the schema's docstring why only one provider populates this today and what it means when absent (confidence/quality falls back to purely multi-source/coverage-based signals, as it already does).

### 3. `confidence.py` / `quality.py` extension

When `native_quality` is present on a source, factor it into the confidence score alongside the existing coverage/gap/agreement signal — a sample with a poor native quality reading should pull confidence down even if there's no multi-source disagreement to detect it otherwise (this is a real capability gap in the current confidence model: it can currently only infer trouble from disagreement or dropout, never from a source's own self-reported quality). Keep this strictly additive: behavior for sources without `native_quality` (i.e. everything in the original prototype's Track A/B tests) must be provably unchanged — add a regression test that reruns the original synthetic fusion scenario and confirms identical confidence output with and without this change present in the code path.

### 4. Tests (`tests/matrix_engine/test_statsports_provider.py` + additions to existing confidence/quality tests)

All gated behind a successful fetch (local override or the live default URL) via `pytest.mark.skipif`/a runtime skip where they need real data, catching network errors as a skip condition rather than a failure; a few (e.g. the anonymization guarantee, given a small inline fixture DataFrame rather than the real file) can and should run unconditionally regardless of network access.

- Schema validation passes on real STATSports output, including the optional `native_quality` column.
- **Anonymization test (must not be skippable)**: given a fixture with a known fake "real" name, confirm it never appears in the adapter's output — this test should use a synthetic fixture, not the real dataset, so it always runs.
- Native sample rate is measured and reported (not assumed).
- Recovered pitch span falls within a plausible football-pitch range; document what happens (a clear warning/flag, not a silent pass) if it doesn't.
- `native_quality` measurably correlates with independently-detectable signal problems (e.g. rows with poor `Quality of Signal`/low satellite count should show higher position jitter than good-quality rows within the same player's trace) — this is the honest substitute for the ground-truth RMSE check Track B could do, since there's no ground truth here.
- Regression test: existing Track A/B tests and the demo script's Scenario 1/2 numbers are unchanged by this change (confirms the `confidence.py`/`quality.py` extension is additive, not a rewrite).

Run the full `pytest` suite before and after; it must stay green with no network access and no local override configured, and pass meaningfully when the source is reachable (whether that's the live default URL or a local override).

### 5. Report addendum

Add a `## Real wearable data — STATSports addendum` section to `matrix_engine/README.md` (or a new `matrix_engine/STATSPORTS_ADDENDUM.md` if that reads better structurally), matching the honest tone of the existing report:

- What was proven: real-vendor raw export parses cleanly into the common schema; native quality signals are usable and measurably meaningful; the lat/lon→pitch estimate produces a plausible pitch span (report the actual numbers).
- What wasn't proven: still zero real fusion validation (no paired optical source exists for this data); the coordinate transform is an unverified estimate, not a calibrated one (no ground-truth pitch corners were available to check it against); this is one club's training-ground/match data, not necessarily representative of other STATSports customers' export conventions.
- Restate the data-provenance caveat plainly: this used real, non-licensed third-party data, handled outside the repo and anonymized by construction; reproducing this requires sourcing your own copy under the same handling constraints, and this is not a green light to treat this specific dataset as a licensed sample.

## Constraints

- Extend, don't rewrite: reuse `matrix_engine/schema.py`, `confidence.py`, `quality.py` structure from the existing package; don't duplicate them.
- Do not modify `analytics/`, `ingestion/`, `roster/`, `statsbomb/`, `app.py`, or any existing test file (same rule as the original task).
- Nothing from the external dataset, or derived from it, may be committed to the repo, or persisted anywhere beyond a disposable, explicitly temporary cache.
- No new dependencies beyond what's already in `requirements.txt` unless genuinely necessary (a geodetic projection can be done with plain trigonometry/numpy; don't add a GIS library for this; reading a CSV directly from a URL needs no new dependency either — `pandas.read_csv` handles `http(s)://` paths natively).
- Anonymization has no disable switch in any code path that produces output.

## Acceptance criteria

- `pytest` passes in full with no network access and no local override configured (all STATSports-data-dependent tests skip cleanly on the resulting fetch failure, everything else — including all 320 prior tests — still passes).
- `pytest` passes in full when the source is reachable — whether that's the live default URL or a `MATRIX_STATSPORTS_SAMPLE_PATH`/`MATRIX_STATSPORTS_SAMPLE_URL` override — with the new tests exercising real behaviour.
- No file from this dataset is left behind in the repo tree or committed to git after a run; any disk cache used along the way is disposable (`tempfile` or equivalent) and documented as safe to delete.
- The anonymization test passes unconditionally and demonstrably proves no real name leaks into output.
- The regression test proves the `confidence.py`/`quality.py` extension doesn't change Track A/B's original numbers.
- The report addendum is written and honest per the structure above.

## Explicitly out of scope

Real fusion validation against ground truth (no paired optical source exists for this data — that still requires a real design partner per `Matrix.md`); a calibrated (survey-verified) coordinate transform; support for any STATSports export format other than the one in this specific file; redistributing or committing the dataset itself.
