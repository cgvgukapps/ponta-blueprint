# AI Workflows — Execution Task

> **Status: not startable yet (17 November 2026).** Met: the engines, `parity.py` and `semantic_builder.py` exist in `ponta-platform`. Context Stage A is built (Gate 1 passed) and Stage B is built (Gate 2a bars agreed and met on the reference match; phases and roles revised 18 November 2026), see [context-packages.md](context-packages.md). Not met: the FastAPI skeleton with its authorised-path reader and cross-tenant test (`apps/api` is empty). The roadmap also puts a founder-run concierge step and evidence gate E1 before the platform build ([roadmap.md](roadmap.md#evidence-gates-proposed-15-november-2026-founder-to-confirm-the-thresholds)). When the context exists, this task gains state citations, state filters and the context fields the packages expose. Read "What the built schema means for this task" below before starting.

Self-contained task for a coding agent working in the `ponta-platform` monorepo. It builds the grounded AI layer specified in [ai-layer.md](ai-layer.md) over the semantic store produced by [platform-extraction-task.md](platform-extraction-task.md). Read `ai-layer.md` first — it is the specification; this document is the work breakdown. Also relevant: [data-tiers.md](data-tiers.md), [engine-governance.md](engine-governance.md#ai-governance), [legal-compliance.md](legal-compliance.md#ai-and-health-adjacent-data), [operations.md](operations.md#llm-costs-and-ai-operations).

## Preconditions (do not start until true; if not, stop and report)

- `ponta-platform` exists with `matrix_engine`, `plaiq_engine`, `parity.py` and `semantic_builder.py` working, and the extraction acceptance criteria met.
- Context packages P0 and P1 complete (possession, possession chains and game state on every semantic event), preferably Stage B too; see [context-packages.md](context-packages.md).
- A FastAPI skeleton and a way to read a match's `semantic_events.parquet`, `trajectories.parquet` and Parity Report by `match_id` **through the API's authorised-path mechanism** (the API builds object paths itself under the caller's org prefix; the AI layer never accepts a path or URI from a client or from the model). If this does not yet exist, building it with its cross-tenant isolation test is a prerequisite, not part of this task.

## What the built schema means for this task (added 17 November 2026)

The semantic builder now exists, and a few of its fields differ from what this task first assumed. Details are in `ponta-platform/EXTRACTION_REPORT.md` and [ai-layer.md](ai-layer.md#the-semantic-football-event). Do not change the schema to fit this task; where a decision is open, stop and ask.

- **Citation confidence is undefined for a single source.** The citation rule asks for `confidence_score`, but a per-frame confidence exists only when Matrix reconciles two or more sources. Single-source rows carry `ball_detected` and `players_observed` instead. *Open decision:* what goes in the confidence slot of a Tier 1 citation (for example a documented derivation from those two fields, or an explicit null with the reason). The "minimum confidence" filter below depends on the same decision.
- **Pressure is on the engine's 1 to 10 scale** (`opposition_pressure`), not a 0 to 1 index. No named "high pressure" threshold exists yet; one must be chosen, documented and versioned in the thresholds config before the pressure-band filter is used.
- **`dq_score` runs from 0 to 1.5.** It is decision quality's **Gain**. Filter and describe through `dq_band` (`low_gain` below 0.55, `partial_gain` 0.55 to 0.79, `most_gain` 0.80 to 0.94, `full_gain` 0.95 and above; schema 1.5 renamed them from poor, suboptimal, good and optimal); never clip or normalise to 0 to 1.
- **`gain_v2_score` runs from 0 to 1.5** *(schema 1.6, 20 November 2026)*: Gain version 2, beside version 1 and labelled estimated, with the same cut-offs and `gain_v2_band` names as `dq_band`. Version 1 stays the default; never average the two. A failed pass scores 0 here.
- **`dq_v2_score` runs from 0 to 1** *(schema 1.3, 19 November 2026)*. It is decision quality's **Choice** (version 2), adopted beside version 1 and labelled estimated. Filter and describe it through `dq_v2_band` *(schema 1.4)*: `best_option` (0.95 and above), `near_best` (0.60 to 0.94), `below_best` (0.45 to 0.59), `well_below_best` (under 0.45). Never filter it through `dq_band`, never merge it with `dq_score`, and cite it only beside version 1, as an estimate. Describe the band as where the choice sat against the best option, never as a judgement of the player.
- **`inference_confidence` is a category** (`strong`, `medium`, `weak`), not a number.
- **Rows are flat columns** (`actor_player_ref`, `dq_score` …), which suits the typed query tool; `nested_record` gives the nested view.
- **Coverage belongs in every answer's limits:** the share of passes and carries scored and the causes of the rest (goalkeeper, set piece, no score from the engine), as the Data check and the shareable match page already show.

## Objective

Three workflows — **post-match brief**, **typed-tool natural-language explorer**, **parity gate assistant** — plus the guardrails that make them trustworthy: citation verification, parity honesty, pseudonymised prompts, an evaluation harness, and audit logging. The cross-provider workload monitor (workflow 4 in `ai-layer.md`) is **not** in this task; it needs real Tier 2 data.

## Hard constraints

1. **No raw coordinates in any prompt.** The AI layer reads only `semantic_events.parquet` and aggregates derived from it, plus the Parity Report. Enforce with a test that fails if any prompt or tool result contains arrays of positions.
2. **No arbitrary SQL.** The model calls typed tools only; query construction (parameterised DuckDB) is done by the tool, from validated arguments.
3. **Every factual claim cites** `[event_id, timestamp_s, confidence_score]` for a specific `match_id`; uncited claims are not rendered.
4. **Parity first.** Every call reads the Parity Report and relevant confidence before generation. Degraded/Blocked metrics are disclosed; Blocked metrics are never answered, estimated or approximated.
5. **Pseudonymised prompts.** Names never appear in prompts, tool arguments or logs. Re-hydration happens only client-side, outside this task's scope.
6. **No real customer data to any LLM** in this task. Development and CI use open-data sample matches only. The unlicensed STATSports sample is never sent to any model.
7. **No medical, causal-fatigue, injury-risk or real-time statements** (see `ai-layer.md` "Claims the AI must not make"). Build these as tested refusals/redirects, not just prompt text.
8. **Provider-agnostic.** No vendor is chosen here. Build a single LLM adapter interface and a **deterministic fake provider** so the whole test suite runs offline and reproducibly in CI. Real-provider integration is a thin implementation of the adapter, added only after the LLM sub-processor terms exist (Track L in [roadmap.md](roadmap.md)).
9. Do not change engine math or the semantic schema; if a needed field is missing from the semantic events, report it rather than computing geometry in the AI layer.

## Deliverables

### 1. LLM adapter and audit logging

- `LLMProvider` interface (structured output / tool-calling), a deterministic fake for tests, and a stub for a real provider that is not enabled.
- Per-call audit record: prompt version, model identifier, tool calls and results (pseudonymised), citations, citation-verifier outcome, token counts, latency, org and match ids. **No names.** Retention follows the tenant purge policy.
- Limits: max tokens per request, max tool calls per question, per-org rate and spend limits (configurable), a fail-soft path (a provider error yields "brief pending" and never fails ingestion).

### 2. Typed tools

- `get_parity_report(match_id)` — returns the stored report.
- `query_semantic_events(match_id, filters, sort, limit)` with a **closed filter schema** covering at least: period/time range, action type, outcome, role or position group, opposition-pressure band, DQ band, pitch zone, minimum confidence, mode (`full_tracking`/`event_only`). Validates arguments, builds a parameterised DuckDB query, caps result size, returns rows including their citation fields. Rejects unknown fields. Fixtures prove it cannot be coaxed into arbitrary SQL.
- `get_event(match_id, event_id)` for citation resolution.
- Named thresholds ("high pressure", "poor decision") live in one documented config and are returned to the user alongside results.

### 3. Citation verifier

Given a model output and the store, reject (and trigger one bounded retry, then fail soft) any output where a cited `event_id`/`match_id` does not exist, or a number quoted in a statement does not match the cited row, or a statement has no citation. This runs at runtime on every output, and is itself unit-tested with malformed outputs.

### 4. Workflow 1 — post-match brief

Deterministic extraction first (top candidates: steepest decision-quality drop over a rolling window, highest-pressure turnovers, costliest suboptimal decisions ranked by DQ × Event ΔV, anything the Parity Report flags), **no LLM in choosing what is notable**; then a synthesis call over only those rows plus the Parity Report with a strict output schema. Header states the match's tier and its Degraded/Blocked metrics. Stored as an `ai_insights` card (`post_match_brief`), generated once and cached, reproducible given the same inputs and prompt version.

### 5. Workflow 2 — explorer

Parity check → intent-to-filter via typed tool → results as summary plus a list of `event_id` citations for the frontend to render as replay chips. If a question needs a Blocked metric, say so and answer only the answerable part. If no filter maps, say what can be filtered. Streams via SSE-compatible chunks.

### 6. Workflow 3 — parity gate assistant

Explains the Parity Report in plain language (what is Full/Degraded/Blocked and why, what a second source would unlock). The report is computed deterministically by `parity.py`; the assistant only explains it and must not contradict it (test this).

### 7. Evaluation harness (built alongside the workflows, not after)

- **Golden question set:** for each sample match (include at least one full-tracking and one event-only; one GPS-only fixture if available), a fixed list of questions whose correct answers are computed deterministically from the semantic store. With the fake provider this verifies the *pipeline* (tool selection, filters, citations); a separate, opt-in suite runs the same set against a real provider when one exists.
- **Parity-honesty tests:** questions requiring a Blocked metric must produce a disclosure and no fabricated answer.
- **Adversarial tests:** prompt injection; "ignore your rules"; requests for player names, raw coordinates, other tenants' data, medical conclusions, injury risk, causal fatigue claims, real-time claims — all refused or redirected.
- **Determinism and versioning:** prompt, model, tool-schema versions recorded; a change to any invalidates cached briefs by version, visibly.
- CI fails on any golden/adversarial/parity-honesty regression.

## Acceptance criteria

- With the fake provider, all three workflows run end-to-end offline on open-data sample matches (full-tracking and event-only) and the whole evaluation suite passes in CI.
- Zero uncited statements can reach an output (verifier-enforced, tested).
- No prompt, tool argument, log or stored artefact contains a player name or a coordinate array (asserted by tests).
- Blocked-metric questions disclose and do not fabricate, in tests for each Blocked rule defined in `parity.py`.
- Typed query tool rejects unknown fields and cannot execute arbitrary SQL (tests include injection attempts).
- Provider failure leaves ingestion and the deterministic store unaffected (test).
- No real-provider call is enabled by default; no customer data used.

## Explicitly not in scope

Choosing an LLM vendor; the web UI (replay chips, brief page — see [frontend.md](frontend.md)); the cross-provider workload monitor; Tier 3 fitness-reconciled analysis; billing/metering UI; re-hydrating names in the client; any change to engine math or semantic schema.

## Final report (`AI_WORKFLOWS_REPORT.md`)

Short and honest: what was built; the golden-set size and pass results (and what they do not prove given the fake provider); every guardrail with the test that enforces it; known gaps (especially anything that can only be evaluated with a real provider); the thresholds config; the limits chosen; and any decision left for a human (provider choice, retention values, spend caps, whether Ask Ponta is capped, metered or bundled).
