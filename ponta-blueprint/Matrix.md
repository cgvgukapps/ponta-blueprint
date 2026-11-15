# Matrix — Unified Tracking Engine

See [README.md](README.md) for baseline assumptions and document index. This document defines **Matrix**, the data engine at the core of Ponta, and its relationship to **Plaiq**, the decision-quality engine.

> **Revised strategy (supersedes the "two products" framing below where they conflict).** Ponta is now planned as **one AI-native application**. Matrix is the **built-in fusion engine inside that app**, not a separately marketed product: it normalises, reconciles and fuses whatever a club uploads into one canonical, quality-scored dataset, and emits a per-match **Parity Report** (Full/Degraded/Blocked per metric). Plaiq's math then turns that dataset into semantic football events that grounded AI agents read. What Matrix does for a given club depends on the data tier it brings — see [data-tiers.md](data-tiers.md) — and the AI built on top is specified in [ai-layer.md](ai-layer.md). The sections below record the original product framing, the evidence, and the limits; the evidence and limits are unchanged and still binding. The public naming question (Matrix/Plaiq inside a one-product app) is open — see [website.md](website.md#launch-positioning-one-product-ai-native).

## Company and product framing

*(Original framing, retained as the record of how the two engines were positioned before the one-product revision above.)* The company is **Ponta**. It took two products to market:

- **Matrix** — the independent layer between a club's tracking providers and their analysis. The main offering, and the strategic priority. Public headline: *"One dataset from every tracking provider."* Public tagline: *"The operating system for football tracking data."*
- **Plaiq** — the IQ analytics engine (off-ball decision quality, pressure, pass option value — everything specified in `ANALYTICS_SPECIFICATION.md` and covered elsewhere in this blueprint). Public line: *"scores the decision, not just the outcome."* Spell out "pass option value" in any public copy; do not use the initials "POV" on pontahq.com. Plaiq ingests **either** Matrix's unified, multi-source-reconciled tracking output **or** a single vendor's raw tracking/event data directly — Matrix is an upstream enrichment layer, not a hard dependency of Plaiq. Public shorthand: *"Plaiq runs on Matrix's output, or on its own."*

This is a deliberate reprioritization from the rest of the blueprint, which was written analytics-first. `competitive-landscape.md` found that Plaiq's core metric overlaps severely with SkillCorner's Game Intelligence product at commercial scale. Matrix is a different bet: instead of competing with tracking/GPS vendors on analytics built from their data, it becomes the vendor-neutral layer that reconciles their data together — a bet that a club's real, unsolved pain (multiple vendors, no common ground truth, manual reconciliation) is more defensible than a second entrant into an already-productized analytics category.

## Vision

Build the industry's first vendor-agnostic tracking data engine that automatically reconciles optical tracking, wearable GPS, and event data into a single trusted dataset for football analysis. Instead of replacing providers such as SkillCorner, TRACAB, Catapult, or STATSports, Matrix becomes the integration layer that makes them work together.

## Problem

Clubs, federations and performance teams collect tracking data from multiple systems, each with different strengths:

- **Optical tracking** provides tactical positioning and team shape.
- **GPS wearables** provide highly accurate physical performance metrics.
- **Event providers** capture actions on the ball.

These datasets are typically stored in separate systems with different timestamps, coordinate systems, player identifiers, and sampling frequencies, requiring analysts to manually align data or build custom internal pipelines. Public framing of the same problem (in the built, not-yet-public site copy): a provider switch breaks multi-year history; systems disagree on threshold-based calls (high-speed, load); analysts become the integration layer. Do not open public copy with "elite clubs" — the ICP in `go-to-market.md` is fragmentation, not the top flight.

## Solution

Matrix ingests tracking data from multiple providers and produces a calibrated, **harmonised** tracking dataset — not a "ground truth" one. That word choice is deliberate: Matrix hasn't been validated against an independent reference (surveyed positions, a controlled ground-truth rig), and two systems can correlate highly while still systematically disagreeing in absolute terms. "Harmonised" (or "canonical") states what's actually been built and is defensible under scrutiny; "ground truth" is a stronger scientific claim than the evidence supports.

Core capabilities split into three distinct layers, worth keeping conceptually separate rather than treating as one blended "reconciliation" step — each is independently useful, independently sellable, and maps onto a different part of the pipeline:

- **Normalisation** — *"these files describe the same concept differently."* Coordinate system, units, sample rate, and schema alignment (e.g. km/h vs. m/s, top-left vs. centred origin). The easy, mechanical layer — already proven across genuinely different real vendors (Track A below).
- **Reconciliation** — *"these systems measured the same phenomenon differently."* Timestamp synchronisation and cross-source player ID matching, where two sources describe the same real event but disagree on when/who. This is where calibration and drift-correction live.
- **Fusion** — *"each source adds what the other is missing."* Combining genuinely complementary signals (e.g. GPS's acceleration precision with optical's team-geometry/positional context) into one combined representation. The least proven layer — see Track B below. (Internal docs may still use the older gloss *"both sources contain information the other doesn't"*; public copy uses the line above.)

Concretely, today: automatic player ID matching and timestamp synchronisation (reconciliation), coordinate/sample-rate normalisation and standardised export format (normalisation), GPS/optical trajectory fusion and missing-frame reconstruction (fusion), plus position confidence scoring and drift/quality detection that cut across all three.

## Supported inputs

| Category | Vendor | Status in `PONTA-PROTOTYPE` today |
|---|---|---|
| Optical tracking | SkillCorner | Adapter exists — `ingestion/skillcorner_io.py` (916 lines), open-data sample match under `sample-data/data/skillcorner/` |
| Optical tracking | Second Spectrum | Adapter exists — `ingestion/second_spectrum_io.py` |
| Optical tracking | Metrica (EPTS export) | Adapter exists — `ingestion/epts_io.py`, `ingestion/metrica_io.py`, `ingestion/metrica_velocities.py`; sample games under `sample-data/data/Sample_Game_{1,2,3}` |
| Optical tracking | TRACAB | Not built — net-new adapter |
| Optical tracking | Hawk-Eye | Not built — net-new adapter |
| Wearables | STATSports | Adapter exists — `matrix_engine/providers/statsports.py`, validated against one real (raw Apex, unpaired, unlicensed) export; see `matrix_engine/STATSPORTS_ADDENDUM.md` |
| Wearables | Catapult | Not built — net-new |
| Wearables | KINEXON | Not built — net-new |
| Wearables | Polar | Not built — net-new |
| Event data | StatsBomb | Adapter exists — `ingestion/statsbomb_io.py` (748 lines) |
| Event data | Opta | Not built — net-new |
| Event data | Wyscout | Not built — net-new |

Roughly half of Matrix's input surface (optical + event) is substantially built and tested already, inherited from Plaiq's existing BYOD ingestion layer. The wearable/GPS half, and true multi-source fusion, is genuinely new engineering regardless of where it's built.

## Outputs

A single enriched tracking dataset containing, per player per frame. **Public copy groups these by tier**, so fused-only fields are not implied to exist under Normalise:

**With Matrix Normalise**
- Player positions in one standard coordinate system, in metres
- Velocity and acceleration
- A quality and confidence score for each frame

**With Matrix Reconcile**
- Unified timestamps across providers
- Players matched across sources

**With Matrix Fuse**
- Source attribution (GPS, optical, or fused)
- Complementary signals combined into one player representation

All of it in a standard export (API/CSV). Internally, keep using "calibrated" where it is accurate; do not use it on the public site (no validated absolute reference exists).

## Value proposition

Reduce hours of manual analyst work while improving the accuracy and consistency of tactical and physical analysis. Rather than forcing clubs into a closed ecosystem, Matrix becomes the vendor-neutral data layer that sits between data providers and downstream analytics — Plaiq's own analytics being the first, but not the only, consumer.

## Product tiers: Normalise → Reconcile → Fuse

**Revised mapping to the data tiers.** In the one-product app these three layers are no longer three things sold separately; they are what Matrix *does* at each data tier ([data-tiers.md](data-tiers.md)): **Tier 1** (single source) → Normalise; **Tier 2** (two unpaired sources) → Reconcile; **Tier 3** (paired match sources) → Fuse. Two refinements from that analysis matter here:

- **Reconcile splits in two.** *With an overlap window* (both systems worn on the same players at the same time) a calibration relationship can be measured. *Without overlap* (a clean vendor switch, or training GPS vs match optical) there is nothing to fit a calibration to; Matrix can match identities, align thresholds and **flag the discontinuity**, but must not claim a continuous history. The "Provider Transition" claim below is therefore only fully defensible when an overlap exists.
- **Fuse has prerequisites that do not exist yet:** a real paired match, ingestion of heart-rate/IMU channels (the STATSports adapter currently ignores them), per-player baselines, and legal review. See [data-tiers.md](data-tiers.md#what-must-be-true-before-tier-3-can-ship).

The commercial-staging description below is retained as written.

The three capability layers above map directly onto a commercial staging that doesn't require solving the hardest technical problem before selling anything — and matches what the Next Steps §4 design-partner ask already does in practice, made explicit here as a named structure:

Public availability labels (founder wording, in the built, not-yet-public site copy) sit beside each tier. They are marketing status chips, not a hosted-product claim:

1. **Matrix Normalise — Available now.** Customer gives one provider's export; Matrix returns a standardised schema, cleaned tracking data, quality scoring, and consistent thresholds. No second data source required, no reconciliation claim being made. Delivery today is founder-run (send an export, we send it back), not a self-serve API. Proven by Track A, and the Track A proof is now a public section on pontahq.com.
2. **Matrix Reconcile — Early access** (FAQ: opening to design partners). Customer has two providers (e.g. surviving a vendor switch, or running GPS alongside optical). Unlocks cross-provider comparison, calibration, and historical continuity across a provider transition. No signed design partner yet.
3. **Matrix Fuse — In development.** Customer has GPS + optical + event data together. Unlocks a combined player representation and richer workload modeling — the hardest, least-proven tier (Track B below), and the one still blocked on a real design partner.

**A concrete use case for tier 2 — "Provider Transition."** A club switching tracking/GPS vendors (a real, recurring event — budget cycles, contract renewals, hardware failures) otherwise gets a hard discontinuity in its historical player-workload data: the old provider's numbers and the new provider's numbers aren't directly comparable. Matrix's pitch here is concrete and easy to understand without any technical background: *"your multi-year player workload history remains analytically continuous through the switch."* This is also a natural moment when a club already has budget and motivation to buy — the switch itself is the trigger, not something Matrix has to create.

## Strategic positioning

Public language is locked in [website.md](website.md). Do not invent a third set of taglines.

- **Tagline:** "The operating system for football tracking data."
- **Headline:** "One dataset from every tracking provider."
- **Independence:** Matrix is independent of every provider. Ponta works alongside the systems a club already uses and treats each provider's data on equal terms. This is now a lead claim, not a buried principle — it is the structural edge against vertically integrated vendors.
- **Product principle:** don't compete with tracking providers. Become the infrastructure that makes every tracking provider more valuable.

Every analytics model — pitch control, xT, physical load, tactical analysis, recruitment models, and AI applications — becomes more valuable when built on a single trusted tracking dataset.

```
Data Sources
─────────────────────────────
SkillCorner · TRACAB · Second Spectrum · STATSports · Catapult · StatsBomb · Opta

        ↓

Matrix — Unified Tracking Engine
─────────────────────────────
Synchronisation · Calibration · Sensor Fusion · Quality Assurance · Confidence Scoring

        ↓

Plaiq & other Football Analytics Consumers
─────────────────────────────
Tactical Analysis · Physical Performance · Recruitment · AI Models · Coach Dashboards · Custom APIs
```

**Long-term vision:** evolve Matrix from a data integration feature into the industry's canonical football data infrastructure — potentially licensed to other analytics vendors, not just consumed internally by Plaiq.

## Why Matrix is prioritized over Plaiq

*(Revised: the question is now moot as a product choice — both engines ship inside one app — but the reasoning still explains why the data engine is the foundation and why Plaiq alone was a weak standalone bet.)*

- **Sidesteps the confirmed competitive overlap.** `competitive-landscape.md`'s single most consequential finding was that Plaiq's core metric is already productized by SkillCorner Game Intelligence at 250+ clients. Matrix is not an analytics competitor to SkillCorner/Catapult/etc. — it's infrastructure that makes their own products more useful together, which is structurally not something any single vertically-integrated vendor is incentivized to build for its competitors' data.
- **Real, validated pain, not a hypothesis.** The manual reconciliation problem (different clocks, coordinate systems, IDs, sample rates across vendors) is an operational cost every multi-vendor club already pays, independent of whether any given analytics metric is differentiated.

This reprioritization introduces new risks that don't exist in the same form for Plaiq alone, and aren't yet resolved anywhere in this blueprint:

- **Legal/ToS exposure is larger, not smaller.** `architecture.md` and `legal-compliance.md` were written around Plaiq being "an enhancement layer on data the customer already licensed, never redistributing across tenants." Matrix's core act — *fusing* multiple vendors' proprietary data into one derived, harmonised dataset — is a materially bigger claim, and no vendor's ToS has been checked for whether this kind of derivative combination is permitted, even for internal use by the licensing customer.
- **Wearable/GPS data is more sensitive than tactical tracking data.** Physical/biometric performance data is more likely to fall under stricter data-protection categories (e.g. GDPR special-category health-adjacent data) than positional tracking alone — a legal review gap, not yet covered by `legal-compliance.md`.
- **Competitive response risk from the vendors Matrix depends on.** Catapult (post-Impect acquisition, already marketing "bridging GPS and tactical reality") and Genius Sports/Second Spectrum are natural candidates to build exactly this reconciliation capability themselves, since they each already sit on one side of the data.
- **Harder, unproven engineering bet.** Plaiq's math (pressure, POV, decision quality) is already built and spec'd. Sensor fusion, drift detection, and confidence scoring across independent sensor streams is a different, less mature engineering domain for this codebase — closer to signal processing/state estimation than anything in `analytics/` today.
- **Model-versioning gap, scoped narrowly.** `engine-governance.md`'s model/formula-versioning discipline (an `engine_version` field per row, patch-vs-material-recalibration policy, never silently rewriting history on backfill) is explicitly scoped to `plaiq_engine` only — it says nothing about Matrix's own calibration models (`sync.py`'s clock model, `fusion.py`'s noise-weighting model). The same pattern should extend to Matrix once it has real calibration data to version, but this isn't an urgent gap while Matrix is still pre-design-partner: there's nothing real yet to version.

None of these block starting a prototype — they're why the prototype is scoped as **validation**, not production build-out (see below).

## Build location & repo strategy

> **Revised:** the validation phase is over. Production work happens in a new `ponta-platform` monorepo where `matrix_engine` and `plaiq_engine` become sibling packages ([architecture.md](architecture.md#monorepo-and-repo-strategy)); `PONTA-PROTOTYPE` is kept, frozen, as the validation record. The one-directional dependency rule below (`matrix_engine` depends on `ingestion`/`roster`, never the reverse) carries into the new repo.

Matrix was built **inside `PONTA-PROTOTYPE`** for the prototype/validation phase, as a new, clearly-separated internal package — not a new repo yet (this repo was renamed from `PLAIQ-PROTOTYPE` once it grew to host both Matrix and Plaiq, rather than just the IQ engine alone):

```
PONTA-PROTOTYPE/
  matrix_engine/            # new — Matrix, UI-agnostic, no Streamlit imports
    providers/               # thin adapters wrapping existing ingestion/* modules
    sync.py                  # timestamp synchronisation
    identity.py               # cross-source player ID matching
    fusion.py                 # GPS + optical trajectory fusion
    confidence.py             # per-frame confidence scoring
    quality.py                 # drift / data-quality detection
    export.py                  # standardised output schema
  analytics/                 # existing Plaiq engine — becomes a *consumer* of matrix_engine's output
  ingestion/                  # existing per-provider parsers — matrix_engine/providers/ calls into these, doesn't replace them
```

Rationale (full trade-off analysis already done in this chat's prior turn, summarized here):

- Reuses ~4,700 lines of already-tested ingestion/roster/identity code (`ingestion/*_io.py`, `roster/build_roster.py`, `statsbomb/identity.py`, `pitch_config.py`) rather than duplicating it in a fresh repo.
- Mirrors the extraction pattern `architecture.md` already planned for `plaiq_engine` — a UI-agnostic package with a clean façade, extracted from the same source tree.
- Matrix is exactly as unvalidated right now as Plaiq was before the competitive teardown — a one-way repo split is a mistake to make before proving the reconciliation approach actually works. `matrix_engine` depends on `ingestion`/`roster`, never the reverse, so a future extraction into its own repo/package (once validated, and if Matrix is sold as a standalone product to non-Plaiq customers) is a mechanical split, not a rewrite.
- Avoids doubling CI/dependency/ops overhead for a solo builder before there's a validated reason to.

## Prototype validation strategy

**The honest constraint:** there is no real GPS/wearable data anywhere in this codebase, and no two genuinely overlapping recordings of the *same* match from two different optical vendors in `sample-data/`. A credible prototype has to work within that, not pretend otherwise. Two validation tracks, both fully runnable offline against data already in the repo:

### Track A — Real cross-provider normalisation

Ingest genuinely different real vendors — Metrica (`sample-data/data/Sample_Game_1/2/3`) and SkillCorner (`sample-data/data/skillcorner/matches/1886347/`) — through their existing adapters, and normalise both into one common Matrix schema (shared columns, coordinate system, units). This proves the normalisation layer works across real, structurally different vendor formats, even though the two matches aren't the same fixture.

### Track B — Synthetic dual-source fusion

Since no real second source exists for the same match, use one real source (Metrica tracking, which has known ground-truth positions at 25 Hz) and synthetically derive a degraded "second source" from it: downsampled rate, injected position noise, random frame dropout, shuffled player IDs, and a timestamp offset/drift — standing in for an independent GPS/optical stream of the same match. Run this through `sync.py` → `identity.py` → `fusion.py` → `confidence.py` → `quality.py`, and validate the fused output against the known ground truth (the un-degraded original). This is a standard way to validate a fusion algorithm's correctness before real multi-vendor access exists — it proves the reconciliation math works, not that any specific real vendor pair reconciles cleanly.

Full task breakdown, deliverables, and acceptance criteria for this prototype are in [matrix-prototype-task.md](matrix-prototype-task.md) — that document is the execution spec to hand to a build agent.

## Explicitly out of scope for the prototype

- Real vendor API integrations or credentials (TRACAB, Hawk-Eye, STATSports, Catapult, KINEXON, Polar, Opta, Wyscout) — all net-new and gated on commercial access this exercise doesn't have.
- Production repo split, API/service layer, deployment.
- UI of any kind.
- Resolving the legal/ToS and GDPR questions raised above — those need a real legal review, not a prototype.
- Performance/scale validation — proving the reconciliation math is correct on sample data, not proving it runs fast on a full season.

## Next steps

1. ~~Run the prototype build described in [matrix-prototype-task.md](matrix-prototype-task.md) inside `PONTA-PROTOTYPE`.~~ **Done** — 320 tests passing, real cross-provider normalisation (Track A) and synthetic dual-source fusion (Track B, 24.5% RMSE improvement over the better single input) both validated. See `matrix_engine/README.md` in `PONTA-PROTOTYPE` for full numbers.
2. ~~[matrix-statsports-adapter-task.md](matrix-statsports-adapter-task.md) — ingest a real STATSports export and wire its native quality signal into confidence scoring.~~ **Done** — 382 tests pass with the source reachable, 365 pass/17 skip cleanly without it. Two findings worth flagging: the quality signal that actually works (`Hacc`) wasn't the one the task specified (`Quality of Signal` turned out to carry no measurable relationship to position error), and PCA — the task's specified orientation estimator — proved unstable on this data (near-isotropic point cloud) and was replaced by a minimum-area bounding rectangle. The `confidence.py`/`quality.py` extension is proven additive (bit-identical output on all prior scenarios), and no name or dataset file leaves the adapter or the repo. Full findings, including what still isn't proven, are in `matrix_engine/STATSPORTS_ADDENDUM.md`. The biggest gap is unchanged: no optical source exists for this session, so this replaced a synthetic *ingest*, not a synthetic *validation* — real fusion validation still requires a design partner.
3. Use the resulting metrics (fusion error reduction, identity-matching accuracy, confidence/drift correctness, and now real-vendor ingestion fidelity) as the evidence base for a go/no-go on further Matrix investment — the same validate-before-build discipline `competitive-landscape.md` applied to Plaiq. **Revised:** the go/no-go on *building the app* is now made (Tier 1 proceeds without waiting on fusion); the open decision is on **Tier 2 and Tier 3**, which remain gated on real data. The validation work below runs **in parallel with the build**, not after it — see [roadmap.md](roadmap.md).
3a. **New: add the Parity Report to Matrix.** `matrix_engine/parity.py` generates the Full/Degraded/Blocked report per match at ingestion (generalising the SkillCorner "Analytics Parity" panel). This is a Tier 1 deliverable and is what makes progressive intelligence honest ([data-tiers.md](data-tiers.md#analytics-parity-report)).
3b. **New: ingest the physiological channels.** Heart rate and the accelerometer/gyroscope columns are present in the STATSports export and currently ignored. Using them (and per-player baselines) is a prerequisite for any Tier 3 or load-based Tier 2 claim, and it triggers the health-adjacent-data legal review in [legal-compliance.md](legal-compliance.md#ai-and-health-adjacent-data).
4. The remaining, unresolved blocker either way: a real design partner with two independently-recorded tracking systems on the same match. **This should be pursued as a two-step ask, not a single all-or-nothing one:**
   - **Step A — lead with what's already proven, not what still needs validating.** Track A (real cross-provider normalisation) is a complete, demonstrable value proposition on its own: hand over one export a club already has (whichever single vendor — optical or GPS — they already use), get back a cleaned, quality-scored dataset in one standard schema. This is the actual pain named in this document's own Problem section (manual reconciliation, different timestamps/coordinate systems), independent of fusion, and it's low-friction — no need for the prospect to already have two overlapping systems, just one export and a willingness to look at the result. **That proof point is built into the website** (`#proof`; the site is not yet public): named Metrica and SkillCorner open sample data, two separate after-normalisation tables, credit line attached. The StatsBomb-open-data "public work" piece in `go-to-market.md` is still not produced and is not on the site.
   - **Step B — only once that trust exists, ask for the harder thing.** Real dual-source fusion validation needs the club's *second* source too (most clubs plausibly already run GPS wearables alongside an optical vendor — but per this document's own note above, that data typically sits with a sports-science/medical department, a different buyer than the analytics contact Step A likely reaches). Winning over that second internal champion is a distinct step, not something Step A's contact can necessarily grant on their own.
   - **The validation protocol to run once real dual-source data exists** — more specific than "does fusion reduce RMSE," since that alone doesn't establish the output is actually usable: (A) **absolute agreement** — does Matrix reduce discrepancy between the sources at all; (B) **rank preservation** — do players keep the same relative workload ranking after harmonisation, since a club often cares about who's highest-load more than the exact number; (C) **threshold agreement** — does a player classified as high-speed/high-load by one system stay classified the same way after harmonisation, since downstream decisions (substitutions, training load) are often threshold-based, not continuous; (D) **longitudinal continuity** — does a provider switch produce less discontinuity in historical trend lines post-harmonisation than the raw switch alone would; (E) **error propagation** — the one most worth not skipping: does reconciliation ever make a specific metric *worse* than either single source alone, not just better on average. A pipeline that improves the mean while quietly degrading a subset of matches or players is a real failure mode a single RMSE number would hide.
   - **A caution carried over from the STATSports adapter finding above:** treat this as validating an already-designed system against real data, not a "fine-tuning pass" — the STATSports work already showed a specified assumption (the `Quality of Signal` metric, the PCA orientation estimator) can turn out wrong on real data even when the surrounding architecture is sound. The `sync.py`/`fusion.py`/`confidence.py` pipeline is a genuine head start, not a guarantee that real dual-source data behaves like the synthetic Track B validation.
   - Revisit the legal/ToS questions above before Step B specifically (combining two vendors' data), not before Step A (single-source normalisation carries none of that exposure) — consistent with `legal-compliance.md`'s general principle of not front-loading legal review before a real trigger exists.

**This is now tracked on the same timeline as the app build, not separately** — see [roadmap.md](roadmap.md), where design-partner outreach and real-data validation run in parallel with engineering from the first phase. **Add a Tier 2 ask to the design-partner search:** an *overlap window* (e.g. old and new GPS worn together for a few sessions) is the cheapest real data that would validate Reconcile, and is a smaller, easier ask than a fully paired optical + GPS match.
