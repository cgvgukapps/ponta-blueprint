# Ponta Engine Governance

See [README.md](README.md) for baseline assumptions and [architecture.md](architecture.md) for where `plaiq_engine` and `matrix_engine` sit in the system. This document covers what's needed to trust the engines' output — and, new in this revision, the AI layer built on it — in production, separate from how it's deployed. The original sections cover `plaiq_engine`; the [AI governance](#ai-governance) section at the end extends the same discipline to Matrix, the semantic schema, and the LLM layer.

## Model/formula versioning and backfill

### The evidence this will happen again

`ANALYTICS_SPECIFICATION.md` already documents multiple formula recalibrations to the shipped prototype — the carry-DQ threshold change from `0.35/0.60/0.85` to `0.55/0.80/0.95`, and an explicitly documented *abandoned* experiment (moving the carry-DQ denominator to end-frame, tried and reverted after an audit). This isn't hypothetical future risk; it's the engine's own track record. Production needs to handle this as a normal event, not a rare emergency.

### What a realistic startup builds for this (proportional, not a full ML model registry)

- Add an `engine_version` field (the `plaiq_engine` package's semver, or a git SHA) to every persisted analytics row (POV, Decision Quality, Movement Value, ΔV). This is a schema decision, not new infrastructure — cheap now, and it's the only way to ever answer "was this score computed before or after we fixed the carry-DQ formula."
- Adopt a two-tier change policy:
  - **Patch-level fix** (a bug, not a methodology change) — recompute silently for affected matches via the existing `process_match` worker job, no version-visible change to the customer.
  - **Material recalibration** (a threshold change, a new component in a formula — the kind of change the spec's own changelog documents) — bump a minor/major version, and treat historical backfill as a deliberate, visible operation: a background job that reprocesses existing matches under the new engine version, tracked to completion, so cross-match comparisons and trend lines (`features.md`'s comparison/trend features) never silently mix two incompatible scoring eras.
- Keep the model changelog discipline `ANALYTICS_SPECIFICATION.md` already has (documenting *why* a formula changed, including rejected alternatives) — just formalize it with version tags so a specific analytics row's provenance is traceable back to a specific documented decision.

This is intentionally lightweight: no separate model-registry service, no A/B testing infrastructure — a version column and a backfill runbook are proportionate to a solo-operator, small-scale product.

## Player-population validity scoping

### The specific risk

The engine's physical constants (`DEFAULT_V_MAX = 9.0 m/s`, `DEFAULT_A_MAX = 5.5 m/s²`, `DEFAULT_REACTION = 0.7s` per `ANALYTICS_SPECIFICATION.md`'s appendix) and the pitch-control/pressure model were tuned against adult professional sample data. `features.md`'s recruitment persona implies youth/academy scouting is a natural use case — but applying adult-pro physical defaults to a 16-year-old's tracking data would systematically distort every downstream metric (pressure, control, POV, Decision Quality) without producing an obviously wrong-looking number. A quietly miscalibrated rating is worse than an obviously broken one.

### What a realistic startup does about it

Scope narrowly and say so, rather than silently mis-serving a use case:

- **v1 explicitly targets adult senior football.** State this as a product boundary, not an oversight — in the product itself (e.g., a visible note on any match/competition that doesn't match this population), not just in internal docs.
- **Youth/academy support is an explicit, separately-scoped v2 feature**, requiring real recalibration work — ideally validated in partnership with an actual academy that can sanity-check outputs against known player development context, not just parameter guesses. The infrastructure is already partway there: `roster/player_db.py` already supports per-player physical-profile overrides, so the extension point exists; what's missing is validated defaults for a different population, not new architecture.
- **Goalkeepers are excluded from Decision Quality scoring at launch**, flagged in the product as "not yet scored" rather than silently producing a misleading rating — the POV/DQ model's counterfactual-passing-option framing wasn't designed around GK distribution decisions.
- **Set-piece situations get the same treatment**: the model's assumptions are built around open-play passing/movement; corner/free-kick decision quality is a distinct problem (arguably closer to what TacticAI's set-piece-specific research targets, per `competitive-landscape.md`) and shouldn't be silently scored by a model not designed for it.

The unifying principle: an explicit "not scored, here's why" is always better than a confidently-displayed number that's quietly wrong — this is the same trust logic behind the DQ badge's explainability design in `frontend.md`.

## Data-quality gate before a match reaches a customer

### The gap

`architecture.md`'s worker section says corrupt data should "fail loudly, not silently degrade," but names no actual mechanism. A mismatched tracking/event file pair, a wrong-competition upload, or a partial parsing failure could currently produce a confidently-displayed, wrong Decision Quality Rating for a real, named player — a severe trust failure for a product whose entire value proposition is credibility.

### What a realistic startup builds, proportional to solo scale

- **An automated sanity-check pass immediately after parsing, before the expensive Tier-1 analytics run**: player count roughly 22, pitch dimensions plausible, event timestamps monotonic, tracking/event time ranges overlapping, no NaN-flood — cheap, fast checks in the same spirit as the existing `test_pitch_config.py`/`test_metrica_coords.py` style tests, just run against real ingested data instead of fixtures.
- **A human-visible preview step in the ingestion UI** — a quick freeze-frame render plus summary stats shown before the match is marked "ready," so an obviously wrong file gets caught by a person looking at it, not just an automated check.
- **Manual spot-checking of the first several matches per new customer or new provider/format combination, done by the founder personally.** This is the classic early-startup "do things that don't scale" answer: full automated confidence in data quality across every provider format isn't achievable on day one, and pretending otherwise is worse than an honest manual QA checklist for the first cohort of matches on any new data path.

## Cross-provider player identity: an explicit non-goal for v1

`features.md`'s cross-match identity gap is scoped to reconciling the same player *within one provider's data* across multiple matches (StatsBomb's stable IDs make this tractable; tracking-only providers need name-based reconciliation). A harder, separate problem — reconciling the same human if a customer connects **two different providers** for overlapping matches (e.g., StatsBomb events and a Second Spectrum upload for the same fixture) — is out of scope for v1. State this plainly as a known limitation rather than attempting a fragile heuristic solution prematurely; it's a rare case for most customers, who realistically standardize on one primary data vendor.

## Construct validity and reliability of Decision Quality (added 15 November 2026)

The AI layer's citation verifier proves that a statement matches a row. It does not prove that the row measures a good decision. The engine's math is specified and tested, but nothing yet shows that Decision Quality agrees with expert judgement, and the website's internal notes record two cautions: the rating bands were tuned on the same match used for the worked example, and the passer is the best option in about 65% of the scored passes. Treat those as open questions until a study answers them.

- **Expert agreement study.** Two or three analysts rate about 100 decisions blind (the pass, the alternatives, no engine score shown). Report agreement with the engine's band and rank. Recruit through design partners; it doubles as a credibility artefact for the discovery calls. Run it on matches the bands were not tuned on.
- **Per-player reliability.** A DQ Rating is the mean of one match's scored decisions. Show the decision count and an interval beside it, and do not rank players from a single match. Check split-half reliability across matches once cross-match identity exists.
- **Notable-event recall.** The brief's deterministic selection should be compared with what an analyst would flag for the same match; record how many of their moments it surfaces.
- **Coverage.** Report the share of events scoreable per match (see [data-tiers.md](data-tiers.md#analytics-parity-report)).
- **Wording until then.** No claim that Decision Quality is accurate or predictive. It is a defined, explainable score.

## AI governance

The AI-native revision ([ai-layer.md](ai-layer.md)) adds three things that need the same "proportionate, not enterprise" governance as the engines: Matrix's own models, the semantic schema, and the LLM layer. None requires a separate registry service at this scale — a version field, a changelog and a test set each.

### Extend versioning beyond `plaiq_engine`

- **Matrix calibration models.** The earlier note that `engine_version` policy covered `plaiq_engine` only is now closed: `matrix_engine`'s clock model (`sync.py`), noise-weighting (`fusion.py`), confidence scoring and the Parity Report logic carry their own version and the same patch-vs-material-recalibration policy. A change to how a metric is classified Full/Degraded/Blocked is a **material** change — it alters what the product claims — and is versioned and backfilled visibly, never silently.
- **Semantic schema.** `schema_version` and `engine_version` are on every semantic event row. A schema change that alters meaning (a new band threshold, a renamed field's semantics) bumps the version and triggers a visible backfill, so insights and cross-match comparisons never mix eras.
- **Prompts, tool schemas and models.** Prompt text, tool schema and model identifier are versioned and stored with every generated insight. A changed brief must be explainable ("different prompt version" vs "different underlying data").

### Grounding as a governed property

Treat "every AI claim traces to a deterministic row" as a tested invariant, not a guideline:

- **Citation verifier** at runtime rejects any output with an unresolvable `event_id`/`match_id` or a quoted number that does not match the cited row.
- **Golden question set in CI**: fixed questions over sample matches with deterministically computed answers; a regression blocks release.
- **Parity-honesty tests**: questions that need a Blocked metric must yield a disclosure and no fabricated answer. **Adversarial tests**: prompt injection, requests for names, raw coordinates, medical conclusions, cross-tenant data.
- The extraction's behaviour-preservation tests (the prototype suite, ~382 behavioural tests plus ~582 parametrised ring-distance cases) are necessary but **not sufficient**: the semantic builder, Parity Report, Plaiq-via-Matrix equivalence and AI layer all need new tests of their own.

### Claim policy for physiological and load-related output

The same principle as "not scored, here's why" in the population-validity section above, applied to the new claims:

- **Degraded/Blocked is mandatory disclosure**, not an internal flag. Physiological analysis on a match with no physiological source is Blocked, and the product says so.
- **Associations, not causes.** Decision-quality-vs-load output is correlational and confidence-banded. Wording such as "fatigue caused", "neuromuscular fatigue detected" or "injury risk" is not permitted in product or AI output. The research basis (weak, largely between-player correlation) is recorded in [roadmap_v2.md](roadmap_v2.md#feature-3-fitness-reconciled-decision-quality-physical-load--decision-timeline).
- **Baselines need history.** A "% of baseline" or ACWR-style figure is shown only for players with enough multi-session history to define the baseline; otherwise the field is absent, not estimated.
- **Adult senior football only (unchanged)**; goalkeepers and set pieces remain `not_scored` in semantic events, so the AI cannot narrate scores that do not exist.

### Cross-provider player identity (updated)

The earlier "explicit non-goal for v1" for cross-provider identity is **partly reversed** by the revised plan: reconciling the same player across two providers is exactly what Matrix Reconcile does (`matrix_engine/identity.py`), and Tier 2/3 depend on it. It remains a Tier 2/3 capability validated only on synthetic data; it stays out of Tier 1 and must not be presented as validated until a real vendor pair exists.
