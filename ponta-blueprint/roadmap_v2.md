# Ponta Roadmap v2 — Post-Market Vision

See [README.md](README.md) for the document index and [roadmap.md](roadmap.md) for the near-term, get-to-market build plan. This document is different in kind, not just in time horizon: **`roadmap.md` is what you commit to building; this document is what you show investors to justify why the company is worth funding beyond the initial wedge.** Nothing here is scheduled work — it's a small set of genuinely ambitious, differentiated capabilities (three for Plaiq, one for Matrix), each stated with the same honesty about prerequisites and risk that the rest of this blueprint uses, because a credible ambitious pitch beats a hand-wavy one with a football-literate or diligence-savvy investor.

> **Revised for the AI-native one-product strategy.**
> - **Phase references in this document** (e.g. "Phase 3/4", "Phase 4/5") point to the *pre-pivot* roadmap. In the rewritten [roadmap.md](roadmap.md) the equivalents are: "customers/usage history exist" ≈ **Phase 3 onward**; "expand" ≈ **Phase 5**; Tier 2/3 real-data work ≈ **Phase 4**.
> - **Feature 3 (fitness-reconciled decision quality) is promoted out of this vision document** into the committed plan as **Tier 3** ([data-tiers.md](data-tiers.md)). Its section below is retained because its research caveat, race-not-moat framing, prerequisites and staged path are still the authoritative statement of how to pitch and build it — and they are binding on the app's claim policy ([engine-governance.md](engine-governance.md#claim-policy-for-physiological-and-load-related-output)). What changed is that it is now the planned flagship, not a post-raise idea; what did **not** change is that it is gated on real paired data, physiological ingestion and legal review, and stays labelled *in development* until then.
> - **Features 1, 2 and 4 remain vision.** Feature 1 (club-tuned model) now has a better foundation: the semantic event store and the AI layer's candidate-option data are exactly the inputs a preference model needs. Feature 2 (predictive team shape) still departs from batch-only architecture and is unchanged. Feature 4 (Calibration Graph) still requires several real pairwise validations first.
> - **Pitch warning added by the pivot:** do not describe the AI layer as the moat. It is table stakes any incumbent can ship; the defensible parts are cross-vendor data, grounded/cited/parity-honest output, and (later) accumulated club-specific and calibration data.

## How to use this document in a pitch

- Frame everything here as **"what becomes possible once we've proven the wedge and have capital/data to invest further"** — not as committed near-term deliverables. Presenting these as Phase 2/3 work (per `roadmap.md`) would be dishonest and would look worse under diligence than a smaller, credible near-term plan.
- Features 1-3 below double as a direct, prepared answer to the hardest question this blueprint has already surfaced against itself: **"SkillCorner already does something very similar — why will you win?"** (`competitive-landscape.md`). The honest near-term answer is the wedges in that document (data-source agnosticism, intent-based scoring, explainability); the honest *long-term* answer is that Plaiq becomes a personalization and simulation platform layered on top of those wedges, which is a materially harder thing for an incumbent to replicate than a single metric. **Feature 4 is a different, Matrix-specific bet** — it answers "why can't a tracking vendor just build this reconciliation layer themselves once they see you've proven it," not the Plaiq/SkillCorner question.
- Each feature below states its **prerequisite gate** from `roadmap.md` — don't skip that in a pitch. Investors evaluating a technical product will probe exactly this ("what needs to be true before you can start this"), and having the answer ready is a credibility signal, not a weakness to hide.
- Not all four carry the same competitive urgency, and the pitch should be honest about that difference: Team Context ML and the Calibration Graph are both genuine, durable moats (a competitor can't copy their way to another club's labeling history, or to accumulated validated cross-vendor calibration relationships); Predictive Team Shape and Fitness-Reconciled Decision Quality are both races against better-resourced players (DeepMind/TacticAI and Catapult/Impect respectively) who could plausibly ship something similar first. Conflating "moat" and "race" in a pitch is the kind of imprecision a sharp investor will catch.

## Feature 1: Team Context ML (club-tuned decision-preference model)

### The pitch framing

Every competing product (StatsBomb IQ, SkillCorner Game Intelligence, Impect) scores decisions against one universal definition of "good." PLAIQ's long-term differentiation is that **the definition of a good decision becomes specific to each club**, learned from that club's own coaching staff over time. A possession-control coach and a counter-attacking coach want different things from the same pass; PLAIQ is the only product positioned to actually learn and encode that difference, because it already computes the full candidate-option set (every teammate's Pass Option Value, every carry's ring alternatives) that a preference model needs as its input.

### Why this is a genuine moat, not just a feature

Every wedge in `competitive-landscape.md` is a *product* difference a competitor could plausibly copy if they chose to invest in it. A club-tuned preference model is an **account-level data moat**: its value compounds with tenure at a specific customer, and a competitor can't copy their way to it — they'd need that specific club's months of accumulated labeling history too. This is the strongest switching-cost argument available in this business, and it's the one most worth leading with when an investor asks about defensibility against incumbents with more data/brand/distribution.

### Why it's technically credible, not just aspirational

This is not a research bet. It's a well-understood ML pattern (preference/reward-model learning, the same family of technique behind human-preference tuning of language models) applied to features PLAIQ already computes. The learning problem is "re-weight already-engineered spatial features per club," not "learn football from raw video" — a lightweight ranking model over existing features, not a deep-learning research program. This is a genuine strength to state plainly in a pitch: it's differentiated *and* buildable by a small team, which is a rarer combination than either alone.

### Prerequisite gate (from `roadmap.md`)

Not buildable meaningfully before **Phase 3/4** — it requires real paying customers generating real usage history, since a per-club model with zero labels has nothing to learn from. The generic, universal DQ score (the whole of `roadmap.md`'s Phase 0-3 product) is the necessary foundation this sits on top of.

### Risks to state honestly, even in a pitch

- **Cold start**: needs a defined threshold before a club-tuned score is shown at all, with the universal score as the default below it.
- **Outcome bias**: coaches labeling after seeing the outcome would quietly reintroduce the "reward what worked" bias that PLAIQ's failed-pass intent-scoring was specifically built to avoid (see `competitive-landscape.md`'s intent-vs-outcome wedge) — the labeling UI must freeze the decision-time frame and hide the outcome.
- **Must not dilute the explainability positioning**: ship as a clearly separate "Style Fit Score" alongside the objective, universal DQ Rating — never conflate the two, or two clubs seeing different numbers for the same event with no visible reason undermines the "no black box" trust argument that the whole product is built on.

### Where it plugs into the existing product

Directly sharpens the "style/role fit filtering" bullet already in `features.md`'s recruitment persona — from a generic decision-making archetype filter into "find players whose tendencies match *our* club's specific tuned model."

## Feature 2: Predictive Team Shape (tactical "what-if" simulation)

### The pitch framing

An interactive tool where a coach repositions a player and sees how the opposing team's shape would realistically respond — moving PLAIQ from *descriptive* analytics (what happened, was it a good decision) to *prescriptive/simulation* tooling (what would happen if we did something different). This is the single most "vision slide"-worthy idea in this conversation, and it should be pitched that way: as the long-horizon payoff of a seed/Series-A raise, not a near-term deliverable.

### Say this plainly in the pitch, don't let it be discovered in diligence

This sits in the same technical territory as Google DeepMind's TacticAI research collaboration with Liverpool FC — already flagged in `competitive-landscape.md` as evidence that the top of this market has research resources a startup cannot casually out-build. Naming that comparison proactively in a pitch is a strength, not a weakness — it shows the team understands the real difficulty of what it's proposing, which is exactly what a technically literate investor is trying to assess. Overclaiming this as "coming soon" and having it unravel under a diligence call with someone who knows the TacticAI paper would be far more damaging than staging it honestly.

### Honest prerequisites (this is what a raise needs to fund)

- **Tracking-data volume**, far beyond what BYOD customers alone would generate early on — realistically hundreds of matches of continuous tracking data, which is also the most commercially guarded data category in the industry. This is a genuine reason a platform-licensed data deal (`features.md`'s "Future evolution" section) matters strategically, beyond just market reach.
- **An applied ML/research hire** — this is not something existing engineering capacity absorbs alongside the rest of `roadmap.md`; it's a distinct workstream requiring different skills.
- **Real-time inference infrastructure** — an interactive "drag and see it respond" UX needs low-latency serving, which is a deliberate departure from this whole blueprint's batch-only v1 architecture (`architecture.md`), not an incremental extension of it.
- **A conscious explainability trade-off** — PLAIQ's core trust positioning is "show your work, no black box" (`frontend.md`); a generative shape-prediction model is inherently harder to explain than a formula-based score. Worth a deliberate design decision (e.g., framing it explicitly as an experimental/exploratory tool rather than a scored output) rather than quietly inheriting the trust expectations set by the rest of the product.

### A staged path that de-risks the pitch

Presenting this as one giant leap is both less credible and less fundable than a staged plan:

1. **v0 — case-based retrieval (buildable pre-funding, no ML research required).** Given a hypothetical repositioning, retrieve real historical frames from already-ingested matches with similar spatial configurations and show what actually happened next. This is a nearest-neighbor lookup over data already collected through normal product usage, not a trained model — a credible, shippable proof of concept that the "what-if" interaction is valuable before committing to the harder version.
2. **v1 — constrained predictive model on a narrow phase of play** (e.g., set-pieces/corners specifically, mirroring TacticAI's own narrower scope rather than attempting full open-play prediction) — the first real learned model, scoped small enough to validate against held-out historical data (does the prediction match what a real defense actually did?) before trusting it in front of a customer.
3. **v2 — full interactive open-play shape simulation** — the complete vision, realistically a multi-year, well-capitalized effort requiring the data partnership and research hire above.

v0 is honestly closeable inside `roadmap.md`'s existing Phase 4/5 without new fundraising; v1 and v2 are the actual "use of funds" story for a raise.

## Feature 3: Fitness-Reconciled Decision Quality (physical load × decision timeline)

### The pitch framing

Overlay match-time physical exertion data — from whatever wearable/GPS system a club already uses — onto the existing Decision-Quality timeline, so a coach or sports scientist can see whether a specific player's decision-making measurably degrades as physical load accumulates within a match. This turns PLAIQ into a bridge between two departments that today work from completely separate tools: sports science (fatigue, load management) and analytics/coaching (decision quality). No other product found in a competitive check does this today, as a named, purpose-built feature.

### This one is grounded in real research, with an honest caveat worth stating in the pitch itself

Multiple peer-reviewed studies confirm mental and physical fatigue measurably impair passing accuracy and decision-making in soccer specifically — this isn't speculative. But the most directly relevant study (GPS-derived external load vs. post-match cognitive/psychomotor decline in professional players) found the correlation was **weak and mostly explained by stable differences between players**, not a strong moment-to-moment causal signal. Stating this caveat proactively in a pitch — "we know the science, and we know it's a real-but-noisy signal, so we're building the confidence-banded version, not the overclaimed version" — is a credibility strength given this whole document's pattern of not overclaiming, in the same spirit as the honest treatment of Predictive Team Shape above.

### Why this is a race, not a moat — say so plainly

Unlike Team Context ML, this is not a category where PLAIQ has a structural head start. **Catapult — the dominant football wearable/GPS vendor — acquired Impect (a named competitor in `competitive-landscape.md`) in 2025/2026**, meaning one company now owns both a wearable-data pipeline and a decision-value metric under the same roof, and their own marketing is already using language like "bridging the gap between GPS and tactical reality." Nothing publicly shipped yet matches this feature exactly, but this is the most time-sensitive competitive signal anywhere in this blueprint — closer to "get there before a well-resourced, vertically-integrated competitor executes their obvious next move" than a defensible long-term moat. If pursued, it may be more realistic to partner with a smaller/independent wearable vendor and ship a narrow version fast than to try to out-build Catapult's full vision.

### Honest prerequisites

- **A new connector category and a new buyer.** Wearable/GPS data (Catapult, STATSports, Polar) is a separate vendor relationship from tracking/event data, and it's typically purchased and owned by a club's sports-science/medical department — not the recruitment/analytics buyer this whole blueprint has targeted so far. This is a second internal champion to win over, not just a new API integration.
- **Careful statistical design to avoid overclaiming causation.** Given the weak, largely between-player nature of the underlying research signal, the visualization needs honest uncertainty framing (confidence bands, not a bold "fatigue caused this bad decision" narrative) — consistent with the confidence-badge pattern PLAIQ already uses elsewhere, not a new design language.
- **A distinct, heightened legal review.** Physiological signals (heart rate especially) sit closer to GDPR's "special category" health data than positional tracking coordinates do — `legal-compliance.md`'s existing framework was written around tracking/event data and would need a specific extension before this ships, not an assumed extension.
- **Honest scoping of "body state."** Most in-match wearable data is *external* load (GPS-derived distance, sprints, accelerations/decelerations, proprietary load indices) — heart rate is inconsistently captured in-play, and true physiological/cognitive markers are almost always pre/post-match tests, not continuous streams. The buildable v1 is "Decision Quality vs. physical exertion," not the fuller "body state" framing.

### A staged path

1. **v0 — retrospective correlation report.** A per-player, post-match report correlating existing DQ events against a club's own exported load data (even just a CSV upload, matching the file-upload-first philosophy already established for tracking data in `architecture.md`), presented with explicit correlation-not-causation framing and confidence bands. No new real-time infrastructure required.
2. **v1 — live synced timeline in Player Performance mode.** The actual vision: the existing DQ timeline chart with a load/exertion overlay sharing the same time axis. Needs a real wearable-data connector and the sports-science buyer relationship, not just a report generator.
3. **v2 — a genuine early-warning tool** ("this player's decisions historically decline once cumulative load crosses X — consider substituting"). Requires enough individualized historical data per player to make a *personal*, not population-level, claim — and should only be pursued once v0/v1 show the correlation is real and useful for specific customers, not assumed upfront given the research caveat above.

### A note on synergy with Feature 1

Worth flagging, not overstating: if a club is already reviewing decisions for the Team Context ML labeling workflow, that same review moment is a natural place to also surface load-state context — "was this decision made under unusually high accumulated load" — helping a coach judge whether a given decision is a true preference signal or an outlier the model shouldn't learn from. A plausible future integration point, not a dependency either feature needs to wait on.

## Feature 4: The Calibration Graph (a network-effect moat for Matrix)

### The pitch framing

Today, Matrix's positioning is validating individual provider relationships — "does source A reconcile with source B." The long-term version is different in kind: treat every tracking/wearable technology as a node, and every *validated* calibration relationship between two of them as an edge, each carrying its own metric, population, sample size, venue, sampling rate, methodology, measured bias/variance, confidence, model version, and validation date. As Matrix accumulates real design partners across different provider pairs, this becomes a genuine **calibration graph for football tracking technologies** — an asset that compounds with every new validated pair, and one a single vendor building a point-to-point converter has no structural reason to accumulate.

### Why this is a moat, not just an architecture choice

The distinction that matters: a competitor could plausibly copy any single Matrix reconciliation feature if they chose to invest in it, the same way `roadmap_v2.md`'s Feature 1 section frames Team Context ML against product-level competitors. But nobody can copy *accumulated validated calibration relationships* without redoing the underlying real-world data collection themselves, pair by pair — the same account-level-data-moat logic Feature 1 already uses for Plaiq, applied here to cross-vendor calibration data instead of per-club preference labels.

### Why this must not be pursued in v1 — say so plainly

Pairwise calibration (validating A↔B, B↔C, A↔C directly, each against real paired data) has to come first and stay the only mode until there's real evidence to build on. The eventual idea — **network calibration**, inferring an untested A↔D relationship from already-validated A↔B, B↔C, C↔D edges without direct paired data for A and D — is real graph-inference territory, and skipping straight to it would mean shipping unvalidated inferred relationships to a customer as if they were measured ones. That's a direct violation of the same "don't overclaim past your evidence" discipline this whole blueprint applies to the RMSE numbers in `Matrix.md`. Direct paired validation must establish each edge before any inference between edges is trustworthy.

### Prerequisite gate (from `roadmap.md`)

Not a v1 idea in any form — this requires several real, validated design-partner relationships across *different* provider pairs to exist first (`roadmap.md`'s Phase 0/3 Matrix design-partner search, repeated multiple times with different vendor combinations), which is itself gated on Matrix clearing its first real dual-source validation at all. This is further out than Team Context ML, which at least only needs one customer's usage history — this needs several, spanning different vendor pairs.

### Risks to state honestly, even in a pitch

- **Cold start is worse than Feature 1's.** A calibration graph with one or two edges isn't yet a graph — it's just the pairwise validation work `Matrix.md` already scopes as the current blocker. The moat only exists once there's enough real coverage to make network inference plausible, which could take years of design-partner accumulation.
- **Each edge has its own validity conditions.** A relationship validated at one venue, sampling rate, and player population doesn't automatically generalize — the edge metadata (population, venue, sampling rate) exists precisely because a calibration relationship measured under one set of conditions may not transfer to another.
- **Depends on Matrix succeeding at the pairwise stage first.** This is explicitly a "what becomes possible after the wedge is proven" idea, not a parallel bet — there is nothing to build here until `Matrix.md`'s own next-steps blocker (a real design partner) is cleared at least once.

## Sequencing relative to `roadmap.md`

| Feature | Earliest credible start | What has to be true first |
|---|---|---|
| Team Context ML | `roadmap.md` Phase 4 | Real paying customers, months of usage history, a designed bias-resistant labeling UX |
| Predictive Team Shape v0 (case-based) | `roadmap.md` Phase 4/5 | A meaningful library of ingested matches across customers to retrieve from |
| Predictive Team Shape v1 (constrained predictive model) | Post-raise, informed by this document | An ML hire, a defined tracking-data acquisition plan, held-out validation methodology |
| Predictive Team Shape v2 (full simulation) | Multi-year, well-capitalized | A platform-licensed data deal at meaningful scale (`architecture.md`'s Future Evolution), real-time serving infrastructure |
| Fitness-Reconciled DQ v0 (retrospective report) | `roadmap.md` Phase 4, opportunistically earlier if a design partner offers wearable exports | A single wearable-data connector (even CSV-based), confidence-banded reporting design |
| Fitness-Reconciled DQ v1 (live synced timeline) | Post-Phase 4 | A real wearable-vendor connector, a sports-science department relationship at a customer |
| Fitness-Reconciled DQ v2 (early-warning tool) | Post-raise, data-dependent | Enough individualized historical load/DQ data per player to support personal (not population-level) claims |
| Calibration Graph (Matrix) | Post-raise, well after `roadmap.md`'s Matrix design-partner blocker clears at least once | Several real, validated pairwise calibration relationships across different vendor pairs — explicitly not v1, and further out than any Plaiq feature above |

## What not to do with this document in a pitch

- Don't present Predictive Team Shape's full vision (v2) as near-term roadmap — stage it, and lead with v0 as proof the team can already ship toward the vision incrementally.
- Don't let the Style Fit Score (Feature 1) get conflated with the objective DQ Rating in pitch materials — an investor who later becomes a customer, or talks to one, will notice the inconsistency, and it undermines the same trust argument this document uses to justify the moat in the first place.
- Don't present Fitness-Reconciled Decision Quality's correlation as causation — the underlying research is real but the effect size is weak and largely a between-player, not within-match, signal. A confident "fatigue causes bad decisions" chart is one pointed question away from unraveling in front of a sports-science-literate investor or customer; the confidence-banded framing is the credible version.
- Don't present the Calibration Graph's eventual network-inference idea as something usable soon, or as validated by the current synthetic 24.5% RMSE result — it depends on several *real* pairwise validations existing first, plural, across different vendor pairs, which is further out than any Plaiq feature in this document.
- Don't skip stating the prerequisites out loud. The strongest version of this pitch is "here's our unfair advantage, here's exactly what has to be true before we can build it, and here's the staged plan that gets us there" — not an unqualified feature list.
