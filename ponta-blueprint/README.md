# Ponta Production Blueprint

A design/laydown exercise for taking the validated **PONTA-PROTOTYPE** (Matrix and Plaiq engines plus a Streamlit app) to a multi-tenant, **AI-native** B2B SaaS product. The website ([pontahq.com](https://pontahq.com)) is **deployed but unannounced** (`noindex`; v2 one-product copy on `main`); the product stack described here (API, workers, semantic store, AI layer, web app) has not been built. This blueprint is architecture, scope, sequencing and risk review, for discussion.

## Strategy as of this revision: one AI-native product

**Ponta is one application.** The earlier blueprint tracked two products — Matrix (the priority) and Plaiq (analytics) — and was written analytics-first, then Matrix-first. Following a product-strategy review, Ponta is now planned as a single AI-native football intelligence application:

- **Matrix is the built-in data engine** inside the app: Normalise → Reconcile → Fuse, turning every club's provider files into one quality-scored, canonical dataset.
- **Plaiq's deterministic math is the spatial engine**: pressure, control zones, pass option value, decision quality, failed-pass inference — computed once at ingestion and compiled into **semantic football events**.
- **Context packages describe the whole match** *(added 17 November 2026)*: possession, game state, team structure, phase of play, relationships, space and match episodes, computed deterministically at ingestion, so the AI is not limited to single actions. Ponta is positioned as a **match context model that the AI explains**; Decision Quality is one field among many. See [context-packages.md](context-packages.md).
- **Grounded AI agents read those events.** They never see raw coordinates, they cite every claim, and they read a per-match **Parity Report** (Full / Degraded / Blocked per metric) before saying anything. See [ai-layer.md](ai-layer.md).
- **Intelligence is progressive, set by the data the club brings** ([data-tiers.md](data-tiers.md)): Tier 1 single source (ships first), Tier 2 two unpaired sources, Tier 3 paired match sources (the flagship differentiation bet — gated on real paired data). The app never blocks on the hardest, still-unvalidated piece (real multi-vendor fusion) in order to deliver on the proven one.

**What did not change.** Engines are extracted, not rewritten. Bring-your-own-data stays the model. Batch (post-match) only. Honesty about confidence is a product principle, not a footnote. Where the new strategy conflicts with an earlier evidence-based limit, the evidence wins unless a decision has been recorded to change it (see [website.md](website.md#launch-positioning-one-product-ai-native)).

**What is validated vs. not (still true, stated here so nothing downstream overstates it):**

- *Validated:* Matrix **Normalise** across two real vendors (Track A); a real STATSports ingest; the Plaiq engine's math and spec; an event-only Plaiq mode. The prototype suite is 964 passing tests (582 of which are one parametrised ring-distance test; the behavioural core is ~382).
- *Built and tested on real open data (15 November 2026):* both engines extracted into `ponta-platform` (1,015 tests), Plaiq-via-Matrix equivalence with the prototype's golden outputs, the Data check (`parity.py`) and the semantic event builder, plus a shareable single-file match page from real engine output. Decision Quality's agreement with expert judgement is **not** established ([engine-governance.md](engine-governance.md#construct-validity-and-reliability-of-decision-quality-added-15-november-2026)).
- *Validated on synthetic data only:* Matrix identity matching, synchronisation, fusion, confidence and drift detection (Track B, including the 24.5% RMSE result).
- *Not validated / not built:* any real two-vendor overlap; ingestion of heart-rate and IMU channels; per-player load baselines; the AI layer (no language model is called anywhere); the API, worker and storage; a production web app (a clickable UI prototype on mock data exists); any customer.

## Baseline assumptions (revised)

- **Business model:** B2B SaaS, multi-tenant, bring-your-own-data.
- **Latency:** post-match / post-session batch only. No real-time claims anywhere — including AI output.
- **Scale:** small initial scale — optimise for speed to market over headroom.
- **Frontend:** React web app (**Ponta Frontend**, [frontend.md](frontend.md)) replacing Streamlit. The framework is undecided: the docs say Next.js, the UI prototype is Vite, and the leaning is a Vite static export; distinct from the public site (**Ponta Website**, [website.md](website.md)).
- **Storage:** Postgres (Supabase) for tenancy, metadata and insight cards; **Parquet in object storage queried via DuckDB** for tracking and semantic data ([architecture.md](architecture.md#canonical-store-and-ai-layer)).
- **AI:** structured, tool-calling LLM use grounded strictly in the semantic event store; pseudonymised prompts; mandatory citations ([ai-layer.md](ai-layer.md)).
- **Repo strategy:** a new `ponta-platform` monorepo; `PONTA-PROTOTYPE` is retained, frozen, as the validation record. ([architecture.md](architecture.md#monorepo-and-repo-strategy)).
- **Team:** solo/bootstrapped — managed services, low ops burden. Biased toward shipping Tier 1 first.
- **Personas:** coaching/tactical, performance/medical staff, recruitment, media — served through the same AI workflows rather than separate product lines.

## Documents

**Start here:**

- [status.md](status.md) — **living handoff note**: current state, repo positions, ordered next steps, open decisions, gotchas. Read first when resuming.
- [roadmap.md](roadmap.md) — the revised, phased plan (extraction → AI workflows → web app, with validation and legal running in parallel, not last).
- [data-tiers.md](data-tiers.md) — Tier 1/2/3, the Parity Report, and exactly what must be true before each tier's headline claim is honest.
- [ai-layer.md](ai-layer.md) — the AI-native design: semantic events, the workflows, grounding rules, pseudonymisation, evaluation.
- [context-packages.md](context-packages.md) — the context packages P0 to P9 (possession to episodes), the package contract, build stages and gates, and the 17 November 2026 repositioning. Stage A committed; the rest planned.
- [platform-overview.md](platform-overview.md) — a dated, self-contained snapshot of the whole platform and the evidence status of each part, written for an independent reviewer. Summarises the documents below; does not supersede them.

**Engines and execution tasks:**

- [Matrix.md](Matrix.md) — Matrix as Ponta's built-in engine: vision, layers, evidence, limits, validation protocol.
- [platform-extraction-task.md](platform-extraction-task.md) — execution spec: stand up `ponta-platform`, extract the engines, add the semantic builder and parity report. **Complete** (see `EXTRACTION_REPORT.md` in `ponta-platform`).
- [context-foundations-task.md](context-foundations-task.md) — execution spec: context packages P0 (foundations) and P1 (possession and game state), measurement first. **Complete** (17 November 2026; Gate 1 passed).
- [context-structure-task.md](context-structure-task.md) — execution spec: context packages P2 (team structure), P3 (relative zones) and P4 (phase of play), validated against reference labels. **Built** (17 November 2026; Gate 2a bars with the founder).
- [ai-workflows-task.md](ai-workflows-task.md) — execution spec: the grounded AI workflows over the semantic store. **Next**, blocked on the API skeleton and on open schema decisions listed in it.
- [matrix-prototype-task.md](matrix-prototype-task.md) — Matrix prototype build spec. **Complete.**
- [matrix-statsports-adapter-task.md](matrix-statsports-adapter-task.md) — real STATSports ingest. **Complete.**
- [prototype-cleanup-task.md](prototype-cleanup-task.md) — prototype tidy-up. **Complete** (see `CLEANUP_REPORT.md` in the prototype repo).

**Strategy and risk:**

- [risks.md](risks.md) — the master risk register, extended for the AI-native pivot.
- [competitive-landscape.md](competitive-landscape.md) — competitor teardown (StatsBomb IQ, SkillCorner Game Intelligence, Impect/Catapult, TacticAI) and what the AI-native angle does and does not change.
- [go-to-market.md](go-to-market.md) — founder-led motion, ICPs, design-partner program.
- [legal-compliance.md](legal-compliance.md) — player data rights, vendor ToS, DPA, plus LLM sub-processor and health-adjacent data.
- [roadmap_v2.md](roadmap_v2.md) — post-market vision for investor conversations (Feature 3 now promoted into the committed plan as Tier 3; the rest remain vision).

**System design:**

- [architecture.md](architecture.md) — system design: engines, canonical store, API/worker/AI split, auth, deployment.
- [engine-governance.md](engine-governance.md) — trusting the engines and the AI in production: versioning, validity scoping, data-quality gate, AI evaluation.
- [operations.md](operations.md) — API governance, data lifecycle, resilience, unit economics including LLM cost.

**Product and UX:**

- [features.md](features.md) — product scope: shared core, AI workflows, persona use, sequencing.
- [frontend.md](frontend.md) — **Ponta Frontend**: Match AI Brief, Pitch Replay, Ask Ponta, Parity onboarding gate.
- [frontend-design-brief-v2.md](frontend-design-brief-v2.md) — the brief for the UI prototype round and the record of fixed design decisions (section 3). Items 5.1 to 5.4 are done; 5.5 (API mapping) is on hold.
- [frontend-design-brief-v1.md](frontend-design-brief-v1.md) — the prompt for an external agent to prototype the dark platform UI (v1; revise after reviewing the prototype). *Superseded by v2.*
- [website.md](website.md) — **Ponta Website**: current built site, and the launch positioning for one AI-native product.
- [website-v2-plan.md](website-v2-plan.md) — the refactor plan for the launch version of the website: decisions, section map, workstreams, launch gates. **Executed**; the launch gates remain open.
- [visual-design.md](visual-design.md) — visual language and brand.

## Source material

Grounded in direct analysis of `PONTA-PROTOTYPE/` — `ANALYTICS_SPECIFICATION.md` (the engine's living spec), `app.py` (the Streamlit UI), `matrix_engine/` and its README/addendum, and `CLEANUP_REPORT.md`. File/line references point into that repo. The website source lives in `Ponta-Website/`.

## Deferred (still genuinely out of scope)

- Which provider(s) to approach for a platform-level data licence and the pricing of a "data included" tier ([features.md](features.md#future-evolution-platform-licensed-data-as-a-second-market)).
- Billing/plans/Stripe, including how LLM-inclusive pricing is structured ([operations.md](operations.md#unit-economics)).
- Real-time/live-match capability.
- A full component-library design system ([visual-design.md](visual-design.md)).
- Video-linked features (clip sync, highlight export) — pitch replay is 2D from trajectories; video is new work.
- **IP protection for `ANALYTICS_SPECIFICATION.md`** — go-to-market suggests publishing parts as a credibility asset, in tension with keeping formulas as trade secrets. Unresolved; decide deliberately.
- **Public naming** of Matrix/Plaiq inside a one-product app — open until launch ([website.md](website.md#launch-positioning-one-product-ai-native)).
- Which LLM provider(s) to use ([ai-layer.md](ai-layer.md)).
