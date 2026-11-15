# Ponta Website

> **Revised for the AI-native one-product strategy.** The site described in "What it contains (as shipped)" below is **built but not yet public**, and it still presents Matrix and Plaiq as two products with a Matrix-first story. That copy is **superseded for launch** by [Launch positioning: one product, AI-native](#launch-positioning-one-product-ai-native) below. Because the site has never been public, there is no live copy to protect and no reason to launch the old positioning; but the site must not launch until the product exists behind it. References below to the site being "live" or "shipped" mean *built*. Updating the `Ponta-Website` repo itself is a separate task, not done as part of this blueprint revision.

See [README.md](README.md) for baseline assumptions, [go-to-market.md](go-to-market.md) for the founder-led motion this site supports, and [visual-design.md](visual-design.md) for how it looks. This document covers **what the built site at [pontahq.com](https://pontahq.com) (deployed, friends-only URL, not announced) says and doesn't say**, and how it is hosted — plus, new in this revision, the launch positioning.

**For the as-built site, the `Ponta-Website` repo is the source of truth** (not yet public). Copy, section order, tone, hosting, and visual language were decided there. Where this blueprint previously specified a smaller, darker, more caveat-heavy page, those earlier decisions are superseded. Approved copy and the full decision log live in that repo's `content/draft.md`; do not invent new public wording here.

## What this is, and isn't

This is the public, unauthenticated commercial site — distinct from **[Ponta Frontend](frontend.md)**, the authenticated client portal where Matrix's and Plaiq's actual tools will live. Ponta Website's job is to make a visitor want to get in touch. Ponta Frontend is where a customer with an account actually does the work. Conflating the two — e.g. bolting marketing pages onto the same Next.js app as the product — is still the wrong split.

`app.pontahq.com` is reserved but **not live**. Sign-in is visible in the footer (and was in the nav) but is **disabled** until the product exists behind that URL. Do not treat a Sign-in link as a true statement until it is turned back on.

## Domain, DNS, and hosting

`pontahq.com` (root domain), registered and DNS-managed on Cloudflare. Ponta Frontend is reserved at `app.pontahq.com`, the API at `api.pontahq.com` (see [architecture.md](architecture.md#9-deployment)) — both on separate hosting from this site.

**The site is served as a Cloudflare Worker with static assets**, not Cloudflare Pages. Founder decision: `wrangler.jsonc` in the website repo points at `dist/` (produced by `scripts/build.sh`). There is no Worker logic — only static files. Cloudflare settings: build command `sh scripts/build.sh`, deploy command `npx wrangler deploy`. The build copies only the files the page loads, so `content/`, `context/`, `scripts/` and `AGENTS.md` are never published.

Still deliberately **not** part of the `architecture.md` product stack (Next.js / FastAPI / Supabase): no shared build pipeline, no database, no per-tenant anything.

## Tone: promote, don't qualify

This is a marketing site. Public copy promotes what Ponta does now **and** what it is building, in confident language. Limitations are discussed personally by the founder, not written on the page. Do not put "can't do", "does not", "not yet", "has not been validated" or similar on the site.

**This is a public-comms rule, not an internal-honesty rule.** `Matrix.md`, `competitive-landscape.md`, and `engine-governance.md` stay evidence-first. The caveats that used to sit on the page (synthetic fusion, no dual-source validation, band thresholds tuned on the same match, passer as best option ~65% of the time) are recorded in the website repo's `content/` (`draft.md`, `plaiq-sample.md`) for founder conversations.

Hard limits that still apply on the page:

- Never call Matrix output "ground truth".
- Never claim validation, accuracy against a reference, or a result that hasn't been produced (including the 24.5% synthetic RMSE figure).
- No "first" / "only" / comparative claims about Plaiq vs SkillCorner, StatsBomb, or Impect.
- No pricing, free trial, "Request a demo", customer logos, testimonials, or product screenshots.
- Plaiq is never described as a Matrix layer.

Status chips (**Available now / Early access / In development**), open-data credit lines, and the form's privacy note stay. Those are labels that keep the page true, not limitation copy.

## Launch positioning: one product, AI-native

*Decided in the strategy revision: positioning moves to one AI-native product; the website's honesty limits are kept. Where the two collide, the limits win.*

### What changes

- **One product, not two.** Ponta is presented as one AI-native football intelligence application. **Matrix is the built-in data engine inside it**; Plaiq's decision-quality analysis is a capability of it. The earlier public rule "Plaiq is never described as a Matrix layer" existed to avoid confusing two separately sold products; under one product it is **retired as a positioning rule** (this is a recorded, deliberate change, not a silent one). Whether "Matrix" and "Plaiq" remain visible names, become feature names, or disappear from public copy is **open** and should be decided at launch with real buyer reactions, not now.
- **Intelligence is shown as progressive.** The site explains that what Ponta can say grows with the data a club brings — one export, two sources, or a paired match — mirroring [data-tiers.md](data-tiers.md). This replaces "Normalise / Reconcile / Fuse" as three separate sold products (the stage names can remain as engine descriptions).
- **The AI is described by what makes it trustworthy**, not by "AI-powered": every statement links to a replay and a confidence; Ponta says what a dataset can't support instead of guessing.

### What does not change (honesty limits, kept)

- Never call Matrix output "ground truth".
- Never claim validation, accuracy against a reference, or a result that hasn't been produced — including the 24.5% synthetic RMSE figure, and including any statement about AI accuracy.
- **No "fused" claim until fusion is validated on real paired data** — *with one recorded exception:* the founder chose the spec's fusion wording for the hero in the staged v2 build ([website-v2-plan.md](website-v2-plan.md), D2), to be re-decided at public launch (gate G6). Fusion has been validated only on synthetic data; "Fuse" elsewhere keeps its *In development* chip.
- No "first" / "only" / comparative claims against SkillCorner, StatsBomb or Impect.
- No pricing, free trial, "Request a demo", customer logos, testimonials, or product screenshots until each is true and approved (screenshots may become appropriate once the product exists — decide then).
- **New AI-specific limits:** no "real-time" (everything is batch); no fatigue-causes-decisions language, no "separates tactical error from neuromuscular fatigue", no injury-risk or medical language; do not describe the AI as the differentiator or moat; do not name an LLM vendor; do not imply the AI has been evaluated beyond what its test set has actually shown.
- The held FAQ questions about data handling stay held until a founder + lawyer answer exists — and now also need to cover LLM processing and pseudonymisation ([legal-compliance.md](legal-compliance.md#ai-and-health-adjacent-data)).

### Hero and sub-hero: proposed vs. usable

| | Proposed in the strategy review | Status |
|---|---|---|
| Hero | "AI-native football intelligence built on fused tracking data." | **Founder override (D2 in [website-v2-plan.md](website-v2-plan.md)):** the spec's wording is used in the staged v2 build despite fusion being validated only on synthetic data. The honesty limit below is not removed; a launch check (G6) re-decides this at public launch. |
| Candidate (claim-safe) | "AI-native football intelligence on one quality-scored tracking dataset." | Usable once Tier 1 ships; **pending founder approval**. |
| Sub-hero | "Ponta unifies your optical, wearable GPS, and event feeds into one quality-scored engine — turning raw 25 Hz telemetry into explainable tactical and physical decisions." | **Partly usable**: "quality-scored", "explainable", "one engine" are supportable; "unifies … feeds" is true only for providers actually supported at launch; "25 Hz" is not true of every source (SkillCorner is 10 Hz); "physical decisions" overreaches — use "tactical and physical insight" and only if GPS physical summaries actually ship. **Pending founder approval.** |

### Availability chips at launch

| Capability | Chip | Condition |
|---|---|---|
| Single-source normalisation, parity report, cited briefs and explorer (Tier 1) | **Available** | Only after the product is live and these work end-to-end |
| Two-source reconcile / workload and vendor-transition monitor (Tier 2) | **Early access** | No chip stronger than this until a real overlap validates it |
| Paired-source fusion and fitness-reconciled analysis (Tier 3) | **In development** | Remains so until real paired data and the legal review exist |

### Launch gates (the site goes public when, and not before)

1. The product behind `app.pontahq.com` exists and Sign in can be enabled.
2. Phase 3 exit in [roadmap.md](roadmap.md) is met (a design partner or customer using Tier 1 on real data).
3. Own privacy policy replaces the Formspree-only note; the FAQ data-handling answers (incl. LLM processing) exist.
4. The open-data licence check for Metrica/SkillCorner (below) is done.
5. Copy re-reviewed against the limits above.

The detailed rewrite of sections, nav and FAQ for the launch version is planned in [website-v2-plan.md](website-v2-plan.md) and executed in the `Ponta-Website` repo.

## What it contains (as built — pre-pivot positioning, superseded for launch)

One long page, anchored sections, in this order:

1. **Hero** — tagline *"The operating system for football tracking data."* Headline: **"One dataset from every tracking provider."** Sub-line: Matrix brings optical tracking, wearable GPS and event data into one harmonised, quality-scored dataset, independent of every provider. One CTA: Get in touch. The hero visual is `PontaFlow.svg` over an isometric pitch outline; there is no data-preview panel in the hero.
2. **What Ponta is** — two products. Matrix is *"the independent layer between your tracking providers and your analysis"*, staged Normalise → Reconcile → Fuse. Plaiq *"scores the decision, not just the outcome"* (off-ball decision quality, pressure, pass option value — never the initials "POV" on the page). Independence band: works alongside every provider; treats each provider's data on equal terms.
3. **The problem** — *"Two systems. One match. Different numbers."* Audience is clubs running more than one tracking system or about to change one — not "elite clubs". Three pains: a switch breaks history; systems disagree on thresholds; analysts become the integration layer.
4. **Provider Transition** — Matrix Reconcile use case, immediately after the problem. *"Designed to keep your multi-year workload history continuous through the switch."*
5. **How Matrix works** — ingest → Normalise (Available now) → Reconcile (Early access) → Fuse (In development). Stages can be adopted one at a time.
6. **Three layers** — tabbed stage. Each layer carries its own "what you get" list, so fused-only outputs (source attribution) sit under Fuse, not under Normalise. Confidence and quality run across all three.
7. **Proof** — Track A, live. Named vendors: Metrica and SkillCorner open sample data, shown as two separate sources (different fixtures — never one mixed table). After-normalisation schema. Credit line required. *"What comes next"* points at Reconcile, not at a "what this doesn't show" caveat.
8. **Plaiq** — decision quality, pressure, pass option value; hexagonal grid; runs on Matrix's output or on its own.
9. **Plaiq worked example** — real scored decisions on Metrica Sample Game 3 (anonymised Player 7). Drawn in code, not a product screenshot. Credit line required.
10. **FAQ** — four buyer questions: which providers, how Ponta fits with existing providers, which parts of Matrix are available, what "harmonised" means. Two questions are **held** until there is a legal answer: *"What do we need to send you?"* and *"What happens to our data?"*
11. **Get in touch** — headline *"Using data from more than one provider, or about to switch?"* Formspree form (`https://formspree.io/f/xgaopwep`): name, email, optional "Which providers do you use?", message, honeypot. Privacy note: Formspree processes the form; *"We'll use your details only to reply to you."*

**Still held, not on the page:** "Public work" (the StatsBomb open-data analysis in `go-to-market.md` has not been produced). Excerpts from `ANALYTICS_SPECIFICATION.md` stay unpublished until the IP-protection question in [README.md](README.md#deferred-still-genuinely-out-of-scope) is resolved.

## Public positioning *(pre-pivot — superseded for launch where it conflicts with the section above)*

These lines are the public product language. Internal docs should not contradict them; they may still be more precise in private.

| Role | Public line |
|---|---|
| Tagline | The operating system for football tracking data. |
| Headline | One dataset from every tracking provider. |
| Matrix | The independent layer between your tracking providers and your analysis. |
| Independence | Ponta works alongside every provider you use. Independent of all of them, Matrix treats each provider's data on equal terms. |
| Plaiq | Scores the decision, not just the outcome. |
| Harmonised | One consistent, quality-scored dataset: every provider's data in the same schema, coordinate system and units, with confidence scores alongside. |
| CTA | Using data from more than one provider, or about to switch? |

**Availability labels** (founder wording, now public):

| Tier | Public status | What that means internally |
|---|---|---|
| Matrix Normalise | **Available now** | Founder-run: a prospect sends one export, we send back a standardised, quality-scored dataset. Not a self-serve API. |
| Matrix Reconcile | **Early access** (FAQ: opening to design partners) | No signed design partner yet; real dual-source validation still outstanding. |
| Matrix Fuse | **In development** | Synthetic Track B only. |

**Providers named on the page**, with status: Matrix reads Metrica and SkillCorner optical tracking and STATSports GPS exports into one schema. Plaiq works with StatsBomb event data; Matrix is being built to bring event data into the same dataset. Other providers: invite the visitor to say which they use. Do not list unbuilt adapters (TRACAB, Hawk-Eye, Catapult, KINEXON, Polar, Opta, Wyscout) as if they were supported.

## What it deliberately does not contain

- **No pricing page, no self-serve signup, no "start free trial" / "Request a demo" button.** Sales is founder-led (`go-to-market.md`). One kind of CTA: a conversation via the form.
- **No Ponta Frontend screenshots.** The Plaiq worked example is a labelled, code-drawn illustration of open-data output, not a product UI.
- **No customer logos, testimonials, or invented results.**
- **No file-upload on the contact form.** The sub-line promises a conversation ("we'll show you what Matrix returns"), not that the visitor should send player data through Formspree.

## Nav

| Item | Target |
|---|---|
| Logo | Top of page |
| What Ponta is | §2 |
| Matrix | Problem through Proof (the whole Matrix run) |
| Plaiq | Plaiq + worked example |
| FAQ | FAQ |
| Get in touch (outlined button) | Contact form |
| Sign in | Visible, **disabled**, until `app.pontahq.com` exists |

Sticky at every width, including phones.

## Open items the live site still carries

- **SkillCorner / Metrica open-data licence terms are unverified.** The page names both vendors and shows derived rows, with a neutral credit line. This was accepted as non-blocking at launch; the actual licence terms still need a real check. Same exposure now covers the Plaiq worked example (Metrica Sample Game 3).
- **No Ponta privacy policy** of our own. The form links Formspree's. `legal-compliance.md` does not yet cover website forms; the on-page promise ("we'll use your details only to reply to you") is a public commitment and should stay true.
- **FAQ data-handling questions held** until a founder + lawyer answer exists. Do not publish "what do we need to send you?" / "what happens to our data?" until that exists.
- **Sign in** stays disabled until the product is reachable.
- **Provider list versus the engine record (open decision, 15 November 2026).** The v2 page lists Second Spectrum among supported providers. Its adapter exists but is tested on synthetic fixtures only; the planning repo's hard rule is that only Metrica, SkillCorner, StatsBomb and STATSports are real, and others are "Coming soon". Either the page changes or the rule is revised with a recorded reason. Not changed here: the website repo owns its copy and ledger.
- **Continuity wording.** The Security and data card says "keep your history" on a provider switch. The FAQ correctly says reconciliation works best with an overlap; the card should carry the same condition.
- **Chips ahead of the product.** "Early access" on the workflows is a recorded decision (O2) and is safe only while the site is `noindex`; tie any removal of `noindex` to gate G2.

## When

Built, not public. Launch is gated on the product and the legal items above (see [Launch gates](#launch-gates-the-site-goes-public-when-and-not-before) and Track V/Phase 3 in [roadmap.md](roadmap.md)). Until then it is not a live commercial surface; outreach runs on warm introductions and direct conversations.
