# Ponta Build Roadmap

See [README.md](README.md) for the document index and baseline assumptions. This roadmap was **rewritten for the one-product, AI-native strategy**. It does not invent new decisions about the architecture; it sequences the work defined in [architecture.md](architecture.md), [data-tiers.md](data-tiers.md), [ai-layer.md](ai-layer.md), [features.md](features.md), [frontend.md](frontend.md), [legal-compliance.md](legal-compliance.md) and [go-to-market.md](go-to-market.md).

The earlier version of this roadmap was Plaiq-first and then Matrix-and-Plaiq-first, with a validation phase that gated the rest. Its structure is replaced; its content is carried forward where still true.

## The sequencing principle

**Ship Tier 1 without waiting for the hard problem; run validation of Tiers 2 and 3 alongside, not after.**

- **Tier 1 (single source)** depends only on things already proven: Matrix Normalise on two real vendors, the Plaiq engine, and a real STATSports ingest. It is buildable now and delivers the whole AI-native experience on one export.
- **Tier 2 and Tier 3** depend on real multi-source data that does not exist yet. Their engineering starts when that data does. Their *validation work* (finding design partners, an overlap window, a paired match) starts on day one, because it has a long, relationship-driven lead time and is the main thing the whole company is uncertain about.

An earlier draft of the new strategy placed real paired-data validation last, after the full app was built. That is rejected here: the build-then-validate order is precisely what this blueprint's competitive and risk work argued against, and it would leave the flagship claim untested for the entire build.

## How to read this

- **Phases, not calendar dates.** This is a solo/bootstrapped build with unspecified hours per week; dates would be fiction. Time claims such as "an MVP in weeks" are not carried into this plan.
- **Two parallel tracks run throughout** (below), not a final phase.
- **Exit criteria, not task checklists.** Detail lives in the linked documents and execution-task specs.
- **What is already done** is listed explicitly so it is not re-planned.

## Already done (not re-planned)

- Plaiq engine prototype, spec and tests; event-only mode.
- Matrix prototype: Track A real cross-provider normalisation, Track B synthetic fusion validation, real STATSports ingest ([Matrix.md](Matrix.md)).
- Prototype cleanup: git initialised, shims removed, kinematic defaults fixed, spec corrected (`CLEANUP_REPORT.md` in the prototype repo). Suite: 964 passing (~382 behavioural).
- Ponta Website built (not yet public) — see [website.md](website.md).
- This blueprint, including the competitive teardown and risk register.

## Track V — Validation and go-to-market (runs from day one, throughout)

**Why a track, not a phase:** it is relationship-driven, has long lead times, and every other decision benefits from its output. It also needs no shipped product to start.

- **Discovery calls and design-partner recruiting** per [go-to-market.md](go-to-market.md): warm intros first; screen every conversation for the Plaiq-style wedge **and** for the Matrix data situation at once.
- **Three specific data asks, in rising difficulty**, each unlocking a tier:
  1. **One export** from any provider (Tier 1 proof of value; also exercises the real-vendor schema assumption, e.g. a paying SkillCorner customer's export vs the open-data format).
  2. **An overlap window** — two systems worn on the same players at the same time for some sessions (e.g. old and new GPS during a vendor switch). Cheapest real validation for **Tier 2 / Reconcile**.
  3. **A paired match** — the same match from GPS + optical (± events). Required for **Tier 3 / Fuse**, plus a second internal champion in sports science/medical.
- **Competitive refresh:** what incumbents have shipped in AI/natural-language features, and what Catapult has shipped from Impect ([competitive-landscape.md](competitive-landscape.md#what-the-ai-native-pivot-does-and-does-not-change)).
- **Validation protocol** from `Matrix.md` §4 (absolute agreement, rank preservation, threshold agreement, longitudinal continuity, error propagation) run as soon as any real paired or overlap data exists.
- **Website launch is gated on the product** (and on the legal items below), not scheduled: see [website.md](website.md#launch-positioning-one-product-ai-native).

**Decision points (replace the old "Phase 0 exit"):**

- **Tier 1:** decided — proceed.
- **Tier 2:** go/no-go on building reconcile productisation when a real overlap window exists and validates (or fails to).
- **Tier 3:** go/no-go on building fitness-reconciled analysis when a real paired match exists, the physiological ingestion is built, and the legal review is complete. A "no" on either does not affect Tier 1.

## Evidence gates (proposed 15 November 2026; founder to confirm the thresholds)

The strategy review found that no build phase waits on customer evidence, although [competitive-landscape.md](competitive-landscape.md) calls the SkillCorner overlap the most consequential finding and asks for it to be resolved before further investment. These gates make that explicit. A gate is a check on spend, not a calendar date.

| Gate | Evidence required | Spend it unlocks |
|---|---|---|
| **E1** | At least 8 discovery calls held, with the wedge question, the "intercepted pass" question and a price anchor asked in each; the answers recorded. | Phase 2 platform build (upload connector, worker, API). Phase 1 engine work is not gated: it keeps the validation record usable whatever the answer. |
| **E2** | At least one prospect has sent a real export and received a Parity Report and a cited brief (the concierge step in Phase 2). | Phase 3 web app. |
| **E3** | A design partner has used a brief on a real match and said what they would pay, or why not. | Any Tier 2 or Tier 3 engineering; any billing work. |

If E1 produces no clear wedge, follow the three outcomes already listed in [competitive-landscape.md](competitive-landscape.md#validation-plan-do-this-before-building-more-product).

## Track L — Legal and compliance (runs alongside, each item gated to a trigger)

Triggers from [legal-compliance.md](legal-compliance.md), restated as gates on the build:

| Gate | Must be in place before |
|---|---|
| Basic ToS (acceptable use, liability) | The first design-partner/pilot uses the product |
| LLM sub-processor terms (no-training, limited retention, DPA listing); accurate pseudonymisation language | **Any real customer data reaches an LLM** (Phase 2 onward) |
| Fixed-scope lawyer consultation (GDPR/UK-GDPR, named-athlete rating framing incl. AI briefs, vendor ToS) | The first paid contract; ideally before the first pilot with real data |
| Services/SOW addendum with IP-retention clause | The first custom-tool engagement |
| Health-data (heart-rate/IMU) legal review | The first ingestion of heart-rate/IMU from a real customer |
| Vendor ToS review for derived/fused datasets | Building Tier 2/3 against real data |
| Medical-claims language and terms review | Shipping any load- or fatigue-linked AI output |
| Lawyer-reviewed DPA, retention policy, purge job implemented | The first paid contract |
| Security questionnaire pack, insurance reconsidered | The first larger/enterprise customer |

## Phase 1 — Platform foundation and engine extraction

**Runs:** immediately, in parallel with Track V.

- Create the `ponta-platform` monorepo; tag `PONTA-PROTOTYPE` as the frozen validation record ([architecture.md](architecture.md#monorepo-and-repo-strategy)).
- Extract `matrix_engine` and `plaiq_engine` with clean façades (`MatrixPipeline`, `MatchAnalysis`); all prototype tests carried over and passing. **Coordinate the `ingestion/`/`roster/` split** so the one-way dependency (`matrix_engine` never depends on `plaiq_engine`) is preserved.
- Implement the **Plaiq-via-Matrix equivalence test** and resolve any differences as recorded `engine_version` changes.
- Build **`parity.py`** (Parity Report) and **`semantic_builder.py`**, each with its own tests. These are the AI bridge and the honesty mechanism.
- Stand up Supabase (Postgres + RLS + Auth), the FastAPI skeleton and object storage, **including the automated cross-tenant access test for Parquet paths**.
- Basic ToS.

Execution spec: [platform-extraction-task.md](platform-extraction-task.md).

**Exit criteria:** both engines run standalone from the monorepo with the prototype suite green; a sample match produces a Parity Report and a semantic-events Parquet file; a deployed (even empty) API + auth skeleton; cross-tenant isolation test passing.

## Phase 2 — Tier 1 pipeline and AI workflows

**Runs:** after Phase 1's exit; the data-quality gate and preview are built with the pipeline, not after it.

0. **Concierge step (before the platform):** run a real prospect's export locally through the engines and send back the Parity Report and a cited brief as a static page, founder-run, as Matrix Normalise already is. It exercises the real-vendor schema assumption, tests whether the brief is useful, and meets gate E2 before auth, RLS and the worker exist.
1. Universal file-upload connector → worker → both Parquet artifacts → slim Postgres row and Parity Report. Tier 1 for optical (SkillCorner, Metrica), event-only (StatsBomb) and GPS-physical-only (STATSports).
2. **AI workflows over the semantic store** ([ai-workflows-task.md](ai-workflows-task.md)): post-match brief, typed-tool natural-language explorer, parity gate assistant — **with the citation verifier, golden question set and adversarial tests from the first commit**, not added later.
3. LLM provider adapter, audit logging, rate/spend limits ([operations.md](operations.md#llm-costs-and-ai-operations)).
4. API governance basics (`/v1`, per-org keys, rate limiting).
5. Resolve explicitly (don't let it default): which provider/model; whether event-only analysis ships in the first release; whether Ask Ponta is capped, metered or bundled.

**Exit criteria:** an uploaded Tier 1 match yields a cited brief and a working explorer; all AI guardrail tests pass; Blocked/Degraded metrics are disclosed rather than answered; LLM sub-processor terms in place before any real customer data is used.

## Phase 3 — Web app and first users

**Runs:** after Phase 2's core exists; UI work can begin against stub data earlier.

- Web app (Next.js in the docs; Vite in the UI prototype, undecided) per [frontend.md](frontend.md): shell + auth + org switching, Match Library with tier/parity badges, Parity onboarding gate, Match AI Brief, Pitch Replay (spike Plotly vs Canvas here), Ask Ponta.
- Deploy; Sentry; uptime monitoring; confirm backups restore.
- 2–3 design partners onto the real product (Track V), including whichever supplies the first real SkillCorner export.
- Lawyer-reviewed DPA, vendor ToS review, retention policy and purge job before the first paid contract.
- Decide website launch timing against the product and legal gates.

**Exit criteria:** at least one design partner or paid customer using Tier 1 end-to-end with their own data and the legal basics actually in place, not just drafted.

## Phase 4 — Tier 2 and Tier 3 on real data

**Runs:** only when Track V has delivered real overlap or paired data — each sub-item gated separately.

- **Physiological ingestion:** heart-rate and IMU channels into the schema; per-player baselines (needs multi-session history and cross-match player identity). Triggers the health-data legal review.
- **Tier 2 / Reconcile productisation:** run the validation protocol on the real overlap window; ship the workload and transition monitor with calibration only where an overlap supports it.
- **Tier 3 / Fuse productisation:** run the validation protocol on the real paired match; build fitness-reconciled analysis with confidence bands and association-only wording ([engine-governance.md](engine-governance.md#claim-policy-for-physiological-and-load-related-output)). Treat this as a race against Catapult/Impect: ship a narrow version fast rather than a broad one slowly.
- Matrix fusion upgrade if the real data shows weighted averaging is inadequate (a state estimator is the documented next step in `matrix_engine/README.md`).

**Exit criteria:** the Tier 2/3 claims on the website and in the app are each backed by a real-data validation result that has been produced and recorded; otherwise they remain labelled *in development*.

## Phase 5 — Expand

**Runs:** after Phases 3–4 prove willingness to pay and real-data results.

- Cross-match player identity (unlocks trends and baselines), player comparison, shortlists and team dashboards.
- Report/PDF export; StatsBomb pull connector with Vault credentials; SkillCorner live pull.
- Opposition analysis; media moment export (needs video/export capability that doesn't exist).
- Additional adapters (Catapult, Second Spectrum, TRACAB, Opta, Wyscout), each gated on real access.
- Platform-licensed data tier; youth/academy support; enterprise security pack.
- Real unit economics, including LLM cost, to set pricing.
- The IP-protection decision on publishing `ANALYTICS_SPECIFICATION.md`, made deliberately before anything is published.

Longer-horizon bets (club-tuned preference model, predictive team shape, calibration graph) remain in [roadmap_v2.md](roadmap_v2.md); the fitness-reconciled bet has been promoted into Phase 4.

## Cross-cutting — runs throughout

- Re-review [risks.md](risks.md) at each phase boundary, before the first design partner, before the first LLM call on real customer data, and before the first paid contract.
- Manual spot-checking of the first several matches per new customer or new provider/format combination, by the founder ([engine-governance.md](engine-governance.md)).
- Monitor FIFPro and similar bodies' positions on player data rights.
- Keep AI evaluation sets current as prompts, models and schemas change.

## What this roadmap deliberately does not do

- **No calendar dates or "weeks to MVP" claims.**
- **Doesn't resolve the open decisions it surfaces** — LLM provider, event-only tier timing, Ask Ponta pricing, public naming of Matrix/Plaiq, and the IP-protection call — it sequences *when* each must be decided.
- **Doesn't treat the 964-test suite as proof the new system is safe.** New tests are required at every new seam.
- **Doesn't assume Tier 2 or Tier 3 will validate.** If real data shows reconcile or fuse does not work as designed, Tier 1 still stands and the claims are withdrawn rather than softened.
