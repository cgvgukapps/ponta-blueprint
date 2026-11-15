# Ponta AI Layer

See [README.md](README.md) for the document index, [data-tiers.md](data-tiers.md) for the Tier 1/2/3 model and the Parity Report this layer depends on, and [architecture.md](architecture.md#canonical-store-and-ai-layer) for where it sits in the system. This document specifies the AI-native part of Ponta: what the AI does, what it is forbidden to do, and how every claim it makes is grounded and checkable.

## The principle: deterministic math, grounded language

Clubs will reject a generic chat box pasted onto a dashboard, and an LLM is the wrong tool for 10–25 Hz coordinate geometry. Ponta splits the work along the line where each is reliable:

- **Deterministic engines compute facts.** `matrix_engine` normalises and quality-scores; `plaiq_engine` computes pressure, control zones, pass option value, decision quality, failed-pass inference. These already exist, are tested, and are explainable by formula.
- **A semantic builder turns those facts into structured events.** Every pass, carry, off-ball run and defensive action becomes one enriched row.
- **The LLM only reads and phrases those rows.** It selects, summarises and explains. It never computes spatial results and never sees raw trajectories.

The product's trust position follows from this: *show your work, no black box* (`frontend.md`). The AI layer extends that position — every statement it makes is a pointer to a deterministic row — rather than undermining it.

## Hard rules (non-negotiable)

1. **No raw coordinates in any LLM context.** Trajectory arrays never enter a prompt. All spatial math runs in `plaiq_engine`; the AI sees only semantic events and aggregates derived from them.
2. **Every claim cites.** Every factual statement in an AI output carries a citation array of `[event_id, timestamp_s, confidence_score]` (plus `match_id`). A statement without a resolvable citation is not rendered; the generator is re-run or the claim dropped.
3. **Parity first.** Every AI call begins by reading the match's Parity Report and the relevant per-frame confidence. Degraded or Blocked metrics are disclosed in the output, never silently substituted or implied ([below](#parity-and-confidence-honesty)).
4. **No arbitrary SQL.** The model calls a small set of typed, sandboxed tools; it cannot write free-form queries against the store.
5. **Pseudonymised prompts.** Names and other direct identifiers never leave Ponta's infrastructure inside a prompt ([below](#pseudonymisation-and-llm-providers)).
6. **No medical claims.** The AI describes workload and decision-quality *associations* for staff review. It does not diagnose, predict injury, or state that fatigue *caused* a decision ([below](#claims-the-ai-must-not-make)).

## The semantic football event

`plaiq_engine/semantic_builder.py` compiles tracking and event data into one row per action. This is the bridge between deterministic math and language. Sections are **conditional on the data tier** — a section the match's data cannot support is absent or `null` with a reason, never zero-filled.

```json
{
  "schema_version": "1.0",
  "engine_version": "plaiq-x.y.z / matrix-x.y.z",
  "event_id": "evt_99482",
  "match_id": "m_…",
  "timestamp_s": 4102.4,
  "minute_display": "68:22",
  "period": 2,
  "actor": { "player_ref": "home_2", "role": "RB" },
  "action": {
    "type": "pass",
    "outcome": "completed",
    "target_player_ref": "home_4",
    "inferred_intended_ref": "home_4",
    "inference_confidence": 0.98
  },
  "spatial_intelligence": {
    "dq_score": 0.51,
    "dq_band": "Poor",
    "best_option_player_ref": "home_10",
    "best_option_dq_score": 0.92,
    "xt_delta": -0.014,
    "opposition_pressure_index": 0.84,
    "pitch_zone": "right_half_space_middle_third",
    "mode": "full_tracking"
  },
  "physiological_context": null,
  "quality_gate": {
    "frame_confidence": 0.96,
    "sensor_drift_detected": false,
    "parity": { "spatial_intelligence": "Full", "physiological_context": "Blocked" }
  }
}
```

Notes that differ from a naïve reading of this schema:

- **`dq_score` is not a 0 to 1 ratio.** It ranges from 0.0 up to 1.5 in both golden matches (Metrica Sample Game 3, SkillCorner 1886347). The values in the example are illustrative. Interpret scores through `dq_band` (Optimal at or above 0.95 for the event-level bands), never by assuming a fixed maximum, and do not normalise or clip in the UI or the AI layer. Confirm band thresholds against `ANALYTICS_SPECIFICATION.md` before writing any prompt or UI that cites a scale.

- **`player_ref` is a pseudonym** (`home_2`), not a name. Names are rehydrated only in the client UI ([below](#pseudonymisation-and-llm-providers)).
- **`spatial_intelligence.mode`** is `full_tracking` or `event_only`. Event-only scores are a different confidence class and are labelled as such in any output; they are never merged with full-tracking numbers on one band.
- **`physiological_context` is `null` unless the match has a physiological source and the fields exist.** When populated (Tier 3, or Tier 2 workload views), its fields are things like rolling acceleration/deceleration load and acute sprint distance. Fields expressed *relative to a baseline* (e.g. "% of baseline") exist **only** when that player has enough multi-session history to define one; otherwise the field is absent, not estimated. These channels do not exist in the engine today (see [data-tiers.md](data-tiers.md#what-must-be-true-before-tier-3-can-ship)), so this section is a target schema, not a description of current output.
- **`engine_version` and `schema_version`** are carried on every row, consistent with [engine-governance.md](engine-governance.md#modelformula-versioning-and-backfill), so a brief can be traced to the exact math that produced it.
- Goalkeepers and set-pieces are `not_scored` with a reason, per the population-validity rules in `engine-governance.md`.

## Storage the AI layer reads

Two stores, deliberately separate (detail in [architecture.md](architecture.md#canonical-store-and-ai-layer)):

- **`semantic_events.parquet`** (object storage, queried via DuckDB) — the *only* thing the AI layer reads for evidence. Precomputed at ingestion so no geometry runs at query time.
- **Postgres** — tenancy, match metadata (including `fusion_tier` and the `parity_report`), roster, and generated insight cards.

`trajectories.parquet` (full 10/25 Hz data) is read **only by the frontend pitch-replay path** through the API, never by the AI layer.

## The three workflows

### 1. Automated post-match / session brief ("zero-click")

- **Trigger:** the ingestion worker finishes writing the semantic store for a match.
- **Step 1 — deterministic extraction.** Aggregations over the semantic events pick the top candidates: e.g. steepest decision-quality drop across a rolling window, highest-pressure turnovers, costliest suboptimal decisions (ranked by existing DQ × Event ΔV), plus anything the Parity Report marks notable. No LLM is involved in choosing *what* is notable.
- **Step 2 — synthesis.** An LLM, given only those selected rows and the Parity Report, writes a one-page brief using a strict structured-output schema.
- **Guardrails.** Each bullet carries its citation array and renders as a clickable pitch-replay chip. A brief header always states the match's tier and the Degraded/Blocked metrics. Briefs produced on event-only or low-confidence data say so at the top.
- **Output.** Stored as an `ai_insights` card (`insight_type = post_match_brief`) with `structured_payload`, so it is reproducible, auditable and not regenerated on every view.

### 2. Natural-language tactical and physical explorer

A coach asks, for example: *"Show me second-half turnovers where our midfielders were under high pressure."*

1. **Intent and parity check.** The agent first reads the match's Parity Report. If the question needs a Blocked metric (e.g. load filtering on a match with no wearable source), it states that up front and offers the answerable part: *"Wearable data isn't connected for this match — filtering by optical pressure and second-half turnovers only."*
2. **Typed query tool.** The model calls `query_semantic_events(filters=…)` with a **closed schema** — fields such as period/time range, action type, outcome, role/position group, pressure band, DQ band, zone, confidence threshold, sort and limit. The tool validates the arguments and builds a parameterised DuckDB query itself. The model cannot supply SQL.
3. **Response.** A short summary plus a playlist of matching events, each a pitch-replay chip rendered from `trajectories.parquet` by the frontend (the LLM never touches the coordinates).
4. **Refusal path.** If a question maps to no available filter, the agent says what it can filter on rather than approximating.

Ambiguous terms ("high pressure", "under load") resolve to **named, documented thresholds** shown to the user (e.g. "high pressure = pressure index ≥ 0.7, per the engine's bands"), not to model improvisation.

### 3. Data-quality and parity gate (onboarding assistant)

When a club uploads data, the assistant explains the Parity Report in plain language: what is Full, what is Degraded and why (frame rate, occlusion, event-only mode), what is Blocked, and what additional source would change that. This is built on the Analytics Parity panel the prototype already had for SkillCorner, generalised to every provider. It is also where the human-visible ingestion preview and the data-quality gate from `engine-governance.md` surface. The assistant explains the report; the report itself is computed deterministically by `matrix_engine/parity.py`, not by the model.

### 4. Cross-provider workload and transition monitor (Tier 2; later)

Built on Matrix Reconcile once real Tier 2 data exists. Behaviour depends on whether an overlap window exists ([data-tiers.md](data-tiers.md#tier-2--two-unpaired-sources)): with overlap, a calibration relationship is fitted and its sample size and population are shown; without overlap, the monitor annotates the vendor transition and does **not** present the two eras as one continuous series. ACWR-style signals appear only with sufficient per-player history and are framed as workload signals for staff review. This workflow is **not** part of the first AI build; it follows the first real Tier 2 pair.

## Parity and confidence honesty

- Every response includes the match tier and a short list of Degraded/Blocked metrics that affected it.
- A claim built on frames below a confidence threshold is flagged, or excluded, with the threshold shown.
- Event-only decision-quality outputs are labelled as a separate confidence class.
- The AI never fills a Blocked metric with an estimate, an analogy from another match, or general football knowledge presented as match fact.

## Claims the AI must not make

These constraints encode lessons from `roadmap_v2.md` (Feature 3) and `legal-compliance.md`:

- **No causal fatigue claims.** The evidence for load → decision-quality decline is weak and largely between-player. Outputs say "associated with", show confidence, and never state that fatigue caused a specific decision or that the system can tell a tactical error from neuromuscular fatigue.
- **No injury prediction or medical advice.** Workload spikes are surfaced for staff judgement. Wording such as "injury risk" is not used by the model.
- **No real-time claims.** Everything is post-match / post-session batch.
- **No cross-tenant inference.** The AI sees only one organisation's data in a given request (row-level security enforced upstream of the model).
- **No named-athlete conclusions presented as fact.** Ratings are shown with their formula and confidence; the legal exposure around named-player ratings is in `legal-compliance.md`.

## Pseudonymisation and LLM providers

- Prompts use pseudonymous references (`home_2`) and roles. Real names are rehydrated **only in the client**, after the response returns, via the tenant's own roster under row-level security — consistent with the `identity_registry` / `person_id` pattern in `architecture.md` §6.
- **Pseudonyms are not anonymity.** A prompt that contains a known fixture, date, minute, role and opposition can often be re-identified by a person with football knowledge. The mitigation is contractual and architectural, not just naming: use providers under a **no-training, zero/limited-retention** agreement, minimise context to the selected events, keep match identifiers coarse where the task allows, and list the provider as a **sub-processor** in the DPA.
- Provider choice is abstracted behind one adapter so it can be changed (or self-hosted) without touching workflows.
- Prompts, tool calls and outputs are logged **without names** for audit and evaluation, under the same retention policy as other tenant data.

## Evaluation: how we know the AI is grounded

Because "the AI said so" is not an acceptable standard for a coaching or medical audience, the layer ships with its own tests, treated like the engine's tests:

- **Citation verifier (runtime).** Every cited `event_id` must exist for that `match_id`, and every number quoted in a bullet must match the cited row. Failing outputs are rejected, not shown.
- **Golden question set (CI).** A fixed set of questions per sample match whose correct answers are computed deterministically (counts, rankings, filters). The agent's tool calls and results are compared against these answers; regressions block a release.
- **Parity-honesty tests.** Questions deliberately asked of matches where the needed metric is Blocked must produce a disclosure and no fabricated answer.
- **Adversarial prompts.** Prompt-injection and "ignore your rules" attempts, requests for names, requests for raw coordinates, requests for medical conclusions — all must be refused or redirected.
- **Version pinning.** Prompt text, model identifier and tool schema are versioned and recorded on every stored insight, so a changed brief can be explained.

## Cost and operations

LLM usage is a new cost line and a new failure mode; it is accounted for in [operations.md](operations.md#llm-costs-and-ai-operations). Briefs are generated once per match and cached as insight cards; explorer queries are bounded (token and tool-call limits, rate limits per org) so a single user cannot generate unbounded spend.

## What this document deliberately does not do

- It does not choose an LLM vendor or model; it constrains how any provider is used.
- It does not specify Tier 3 (fitness-reconciled) AI behaviour beyond the claim limits above, since that depends on data and validation that do not exist yet.
- It does not promise a delivery date.
