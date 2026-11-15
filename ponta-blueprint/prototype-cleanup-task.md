# Prototype Cleanup — Execution Task

> **Status: complete.** Executed in `PONTA-PROTOTYPE` (8 commits; see `CLEANUP_REPORT.md` there). The suite went from 382 to 964 passing, of which 582 are one new parametrised ring-distance test; there was one intentional behaviour change (kinematic defaults aligned to 9.0 / 0.7). Retained as the as-executed record. Superseded as the "next step" by [platform-extraction-task.md](platform-extraction-task.md), which starts from this cleaned baseline. One cleanup item remains open and should be decided during extraction: the unused `prev_event` parameter of `statsbomb/identity.resolve_identities`.

This is a self-contained task description for a coding agent working inside `/Users/cicerovieira/PONTA-PROTOTYPE`. It doesn't assume the agent has any other context — everything needed is below or in the files referenced. If anything is ambiguous, `/Users/cicerovieira/PontaPlan/ponta-blueprint/features.md` (Part 1's prototype audit) and `engine-governance.md` have the fuller reasoning behind several items below.

## Context

`PONTA-PROTOTYPE` is a well-tested analytics/ingestion codebase (382 passing tests as of this writing — confirm with `pytest -q` before starting) that's about to enter `roadmap.md`'s Phase 1 ("Foundation"): extracting `analytics/`, `ingestion/`, `roster/`, `statsbomb/`, and `pitch_config.py` into an installable `plaiq_engine` package with a clean `MatchAnalysis` façade (`architecture.md` build order, step 1). **This task is explicitly not that extraction** — it's the tidying pass that should happen first, so the extraction starts from a clean, consistent, version-controlled codebase instead of inheriting clutter, known bugs, and inconsistent conventions into the new package structure.

This task also isn't a rewrite or a redesign — every change should be small, reviewable, and behavior-preserving unless a specific item below says otherwise (and where it does, the reasoning must be documented, since a behavior change to the engine matters for `engine-governance.md`'s upcoming `engine_version` numbering).

## Objective

Leave the repo: (1) under version control with a clean history, (2) free of dead/orphaned files, (3) internally consistent in its import conventions, (4) free of the specific code/spec mismatches already flagged in `features.md`'s audit, and (5) documented well enough that a new agent or collaborator can get oriented without reverse-engineering `app.py`.

## 0. Version control safety net — do this first, before any other change

There is currently **no git repository at all** in `PONTA-PROTOTYPE`. Before touching anything else:

1. `git init`, add a `.gitignore` covering at minimum: `.venv/`, `__pycache__/`, `*.pyc`, `.pytest_cache/`, `out/`. Decide explicitly (and note the decision in your final report) whether `sample-data/` (~1.2GB) belongs in git, in git-lfs, or should stay untracked with `scripts/download_skillcorner.sh` as the documented way to (re)fetch it — don't just silently commit or silently exclude it without saying which you chose and why.
2. Commit the current state as-is (`chore: initial commit of existing prototype state`) **before** making any cleanup changes, so every change below is a reviewable diff against a known baseline, not bundled into one opaque commit.
3. Make the rest of this task's changes as a small number of logically-grouped commits (e.g., one for the shim removal, one for the engine-mismatch fixes, one for orphaned-file removal, one for docs) rather than a single giant commit — this is explicitly for reviewability, since these changes touch several unrelated concerns.

## 1. Remove the root-level backward-compatibility shims

Eleven tiny files sit at repo root, each just a 1-2 line re-export shim (e.g. `build_roster.py` is literally `from roster.build_roster import *`): `build_roster.py`, `epts_io.py`, `genius_events_io.py`, `hex_analytics.py`, `hex_value.py`, `metrica_io.py`, `metrica_velocities.py`, `pass_option_value.py`, `pitch_control.py`, `player_db.py`, `second_spectrum_io.py`, `statsbomb_io.py`. Everything at module load time already imports the proper qualified path (`from analytics.hex_value import ...`, etc.) — but a handful of **function-body-local** imports still use the old flat form, which is exactly the inconsistency to fix before this becomes a packaged dependency:

- `analytics/hex_value.py` — lines ~386 and ~452: `from hex_analytics import hex_to_center` → should be `from analytics.hex_analytics import hex_to_center`.
- `app.py` — lines ~646 and ~3707: `from player_db import (...)` → should be `from roster.player_db import (...)`.
- `preprocess.py` — line ~169: `from genius_events_io import load_genius_events_game` → `from ingestion.genius_events_io import load_genius_events_game`; line ~188: `from build_roster import write_secondspectrum_roster` → `from roster.build_roster import write_secondspectrum_roster`.

After migrating those call sites, grep the whole repo (excluding `.venv/`) for any other reference to the 11 shim module names to make sure nothing else depends on them, then delete all 11 files. Full test suite must stay green.

## 2. Resolve the known engine mismatches from `features.md`'s audit

These four were flagged by name in `features.md` but not yet fixed — each needs an actual decision + fix, not just a note:

- **Kinematic default inconsistency.** `analytics/pass_option_value.py` defaults to `v_max=8.0, reaction_time=0.2` in a couple of places (around lines 206-208 and 320-322), while the canonical defaults used everywhere else in the same file (around lines 1080-1089, 1342, 1369) and in `analytics/pitch_control.py`'s `DEFAULT_V_MAX`/`DEFAULT_REACTION` (lines ~45/47) are `9.0`/`0.7`. Determine which is actually intended (9.0/0.7 matches `ANALYTICS_SPECIFICATION.md`'s documented appendix values, so it's the likely canonical one, but verify — don't just assume), fix the outlier(s), and add/extend a test that would catch this kind of drift in the future.
- **Ring-distance formula.** `ANALYTICS_SPECIFICATION.md` (line ~84) writes hex ring distance as `max(|Δq|, |Δr|, |Δq+Δr|) / 2`; the actual code (`statsbomb/sb_dq.py` line ~235, `analytics/pass_option_value.py` line ~390) computes `(|Δq| + |Δr| + |Δq+Δr|) // 2`. These two forms are the standard axial/cube hex-distance identity and *may* be exactly equivalent — verify this rigorously with a property-based or exhaustive small-range test (don't assume either way), and then either (a) if truly equivalent, update the spec prose to state the form actually implemented so a future reader isn't misled into thinking there's a bug, or (b) if you find a real edge case where they diverge, fix the code and document why. Either outcome must be written up explicitly in your final report — "investigated and confirmed equivalent" is a valid, useful outcome, not a non-answer.
- **Dead/unused constants.** Run a static-analysis pass (e.g. `ruff check` or `pyflakes`, installing as a dev dependency if not already present) across `analytics/`, `ingestion/`, `roster/`, `statsbomb/`, `pitch_config.py` and remove genuinely unused constants/imports/functions it finds — not just the ones anyone happened to notice by eye. List what was removed in the final report.
- **Outdated trend-line prose.** `ANALYTICS_SPECIFICATION.md` §5.1 (~line 366) and §7.4 (~line 840) describe the off-ball/DQ trend-line design ("mean + EWMA + per-half + max spline"). Compare this prose line-by-line against the actual implementation in `app.py` (roughly lines 4014-4432, which includes inline comments describing the current design) and correct any parameter or behavior the prose still describes from an earlier version.

## 3. Remove orphaned artifacts

- **`.video_cache/`** — 7 `.mp4` files; nothing in the current codebase reads or writes this directory (confirmed in `features.md`'s audit). Remove it.
- **`out/matrix_demo/`** — regenerable output from `scripts/matrix_demo.py`; should not live in the working tree/be tracked in git (covered by the `.gitignore` in step 0), and the existing generated copy should be deleted from the working tree.
- **`statsbomb/tests/`** — an empty directory (confirmed zero files) left over from an earlier test-layout convention; the real, populated tests live in `tests/statsbomb/`. Delete the empty one.

## 4. Fill the missing repo hygiene basics

- **Add a root `README.md`** — there isn't one today. At minimum: what this repo is (link to the blueprint's framing), how to set up the environment (`.venv`, `requirements.txt`), how to run the test suite, how to run `app.py`, and a short map of the top-level packages (`analytics/`, `ingestion/`, `roster/`, `statsbomb/`, `matrix_engine/`) and what each owns.
- **Pin `requirements.txt`.** It currently uses open-ended `>=` bounds for every dependency. Generate exact pins from the current working `.venv` (e.g. `pip freeze` filtered to direct dependencies, or `pip-compile` if you introduce it) so Phase 1's engine extraction starts from a reproducible environment, not "whatever the latest version happens to resolve to on a given day."
- **Root-level `MATRIX_PROTOTYPE_TASK.md` / `MATRIX_STATSPORTS_TASK.md`** — these are near-duplicate copies of `ponta-blueprint/matrix-prototype-task.md` and `matrix-statsports-adapter-task.md`, left in place as an as-executed record. Keep them (they're a legitimate historical record of what was actually run), but add one line at the very top of each stating plainly that the blueprint copies are canonical going forward and this is a historical snapshot — so a future reader doesn't wonder which one to trust or edit.

## Constraints

- Do not perform the `plaiq_engine` package extraction itself, do not touch `matrix_engine/`'s internals beyond the shim-import fix in step 1, and do not start any Supabase/FastAPI/frontend work — all explicitly out of scope, covered by separate tasks.
- Every change must keep `pytest` fully green (382 passed, 0 failed as of this writing — confirm the exact current count yourself before starting, since it may have changed). If a mismatch fix in §2 changes a formula's actual output, that's a real behavior change — call it out explicitly and loudly in the final report; do not fix it silently.
- Small, reviewable commits per concern (see §0), not one large diff.
- Match existing code style/conventions already established in `ingestion/*_io.py` and `analytics/*.py` (docstring style, module layout) rather than introducing a new convention.

## Acceptance criteria

- Git repository initialized, sensible `.gitignore` in place, working tree clean, cleanup changes split into logically-grouped commits.
- All 11 root-level shim files removed; the 6 identified function-local call sites migrated to qualified imports; a repo-wide check confirms nothing else references the old flat module names.
- Kinematic defaults consistent across `analytics/pass_option_value.py` and `analytics/pitch_control.py`, with the choice documented.
- Ring-distance formula/spec discrepancy explicitly investigated and resolved (code fixed, or spec prose corrected to match reality) — not left as an open question.
- Dead-code/constants pass completed via a static-analysis tool, with findings listed in the final report.
- Off-ball/DQ trend-line prose in `ANALYTICS_SPECIFICATION.md` matches the current `app.py` implementation.
- `.video_cache/`, `out/matrix_demo/`, and the empty `statsbomb/tests/` are removed.
- Root `README.md` added; `requirements.txt` pinned to exact versions.
- `pytest` passes in full at the end, with the final pass count reported and compared to the starting count.

## Explicitly not in scope

The `plaiq_engine` package extraction and `MatchAnalysis` façade (`architecture.md` step 1 — a separate, later task); any Supabase/FastAPI/API work; any frontend work; any change to `matrix_engine/`'s internal logic (already clean and tested per `Matrix.md` — leave it alone beyond the one shim-import fix noted above); performance/scale work; anything requiring new vendor credentials or network access.

## Final report (`CLEANUP_REPORT.md`, repo root)

Short and honest, matching this blueprint's existing tone (see `risks.md` for the style to match):

- What was removed/fixed, with before/after specifics for each item in §1-4 (not just "done").
- The actual resolution of the ring-distance investigation and the kinematic-default decision, including reasoning.
- What the dead-code pass found and removed (or a note if it found nothing beyond what was already known).
- Final test count vs. starting test count, and confirmation of zero behavior regressions outside the explicitly-documented fixes in §2.
- Any decision left genuinely open for a human to weigh in on (e.g. the `sample-data/` git-tracking question in §0, if it wasn't clear-cut).
