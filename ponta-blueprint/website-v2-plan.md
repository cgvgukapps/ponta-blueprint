# Ponta Website v2 — Refactor Plan

See [website.md](website.md) for the standing rules and launch positioning, [ai-layer.md](ai-layer.md) and [data-tiers.md](data-tiers.md) for what the product can actually claim, and the `Ponta-Website` repo (`index.html`, `content/draft.md`, `AGENTS.md`) for the current build. This document turns the strategy-review feedback for the website into an executable plan. **Planning only — nothing in the website repo has been changed.**

## Current state (verified, not assumed)

- `pontahq.com` is **deployed and reachable** (HTTP 200) and serves the old v1 page ("The operating system for football tracking data"). Only friends have the URL, but there is **no `noindex`** and `robots.txt` is Cloudflare's default, so it can be indexed. *(This corrects the blueprint's earlier "not yet public" wording: it is deployed but unannounced.)*
- The repo is one static page: `index.html` (~101 KB), vanilla CSS split under `css/sections/`, three small JS files, no framework, no build step. `scripts/build.sh` copies a fixed file list into `dist/`; `wrangler.jsonc` serves `dist/` as a Cloudflare Worker with **no Worker code**. The Cloudflare build/deploy commands suggest a push to the production branch deploys.
- Contact form posts to Formspree; Sign in is disabled (the file carries restore instructions pointing at `https://app.pontahq.com`).
- `AGENTS.md`'s synced block is the **old** `website.md`, and its hand-written section still says "Ponta builds two products". Both need updating; the working tree already has uncommitted changes to `AGENTS.md` and `context/visual-design.md`.
- **Open-data licences are still unverified.** `sample-data/` has no licence file and no licence text in its README. This now blocks launch (the spec adds attribution footnotes and a `/terms` page).

## Decisions recorded (founder, this session)

- **D1 — Live site:** edit the deployed site in place; only friends have the URL. Low-cost safeguards I will apply anyway: add `noindex` until launch, and do the work on a branch with a preview deployment so half-built pages are never what friends see. (Confirm Cloudflare preview behaviour for non-production branches during Phase A.)
- **D2 — Hero uses the spec's wording** ("fuses … at the frame level", fatigue-adjusted DQ). **This overrides the earlier decision to keep the website's no-unvalidated-claims limit for the hero.** It is recorded as a deliberate founder override, not a silent one. *Consequence:* the page makes a present-tense fusion claim that today rests only on synthetic data. *Safeguard:* a launch check (G6) re-asks this question when the site goes public.
- **D3 — Worked example:** ship the spec's "Fused Insight Card" **with its numbers, labelled illustrative.** *Defaults I apply, consistent with your earlier Tier 3 decision (association-only, no injury claims):* the label is persistent and adjacent ("Illustrative example — not real match data"), not fine print; the AI-synthesis sentence is worded as an association ("consistent with elevated load") and **drops** the causal claim ("driven by neuromuscular fatigue rather than positional error") and the injury-prevention recommendation ("to prevent soft-tissue overload"). The existing **real** worked example (Metrica Sample Game 3) stays as the honest proof. The hero mock is illustrative under the same labelling rule until a real screenshot exists.
- **D4 — Data-use commitment:** make a **narrow** public commitment — *"Your raw data and club-specific models are never used for any other club."* Do **not** advertise pooled/anonymised calibration statistics on the site; that is a separate, opt-in matter reserved for later, and the sentence needs lawyer review so the DPA matches it exactly.

## How each part of the feedback is handled

**Adopted as written**
- One AI-native product; Matrix presented as the engine inside it, with the three-tier Foundation section.
- The Parity callout ("Ponta never guesses… Full / Degraded / Blocked") — the strongest honest differentiator in the feedback.
- Sitemap `/`, `/security`, `/privacy`, `/terms`; nav (Intelligence · Fusion Engine · Security & Data · Sign in · pilot CTA).
- Pilot form with qualifying fields; real product screenshots once the app exists; Cloudflare Worker deployment; OG metadata; licence verification and attribution.

**Adopted with changes (defaults — tell me if you disagree)**
- **Problem section:** "Optical AI is blind to fatigue" and "AMS co-pilots run on flat summaries" assert things about other products I have not researched, and the comparative-claim rule stands. Rewrite as statements about the club's own data situation (sources that don't talk to each other; load data without tactical context and vice versa; a vendor switch breaking history). *Note:* the hero sub-headline's "legacy tools bolt chat assistants…" sentence is kept by D2; flagged as a residual risk below.
- **Workflow B example:** lead with a tactical query that works on single-source data; keep the ">110% deceleration load" query but mark it as a Tier 2/3 capability.
- **Workflow C:** the engine does not weight decision quality by wearable strain, and "adjusted" implies causation. Name it "Decision Quality"; describe load context as a separate in-development capability.
- **Matrix section:** name only providers supported at launch (not Catapult, Hawk-Eye, TRACAB — unbuilt adapters); "history never breaks" becomes "designed to keep history continuous" with the overlap condition in the FAQ; Reconcile and Fuse keep their *Early access* / *In development* chips (a truth label, not a staging disclaimer — "purge all labels" applies only to what has shipped and been validated).
- **Trust section:** "single-tenant" is false — use "isolated per-organisation workspaces with row-level security"; "Zero-PII" → "names are replaced with pseudonyms before AI processing, and restored in your browser" (pseudonymisation, not anonymity); vendor-switch line carries the overlap condition; D4 wording for the data-use commitment.
- **Pilot CTA:** "Request a pilot" (matches the design-partner programme) rather than "Interactive sandbox"; a hosted sandbox is out of scope until a demo environment is a deliberate product decision. The earlier "no Request-a-demo button" rule lifts at launch because a product exists, but not before.
- **Form backend:** stays Formspree (the repo's rule is no backend), now covered by Ponta's own `/privacy` naming Formspree as a sub-processor, with a data-minimisation note since it collects more personal data than v1. Revisit only if volume or compliance demands it.
- **Demo component:** vanilla JS/SVG from open data in the style of the existing code-drawn SVGs, not React (the repo's guardrail).
- **Sign in:** target is `https://app.pontahq.com` (the spec's "://pontahq.com" is garbled and would loop), enabled only when the app is reachable.

**Not adopted**
- Naming unbuilt providers as unified; "zero black-box hallucinations" (unprovable); "match-winning" (a performance claim); "solves what legacy co-pilots miss" (unresearched comparative claim); "single-tenant"; hosted interactive sandbox.

## Section map: v1 → v2

1. **Hero** — new eyebrow/headline/sub-headline (spec wording per D2); CTAs "Request a pilot" and "Explore a brief ↓"; keep the founder-supplied `PontaFlow.svg` hero until a real screenshot exists; illustrative product mock per D3 if used.
2. **What Ponta is + The problem + Provider Transition** → one **Problem** section (three rewritten problems).
3. **Plaiq** section → **Intelligence** (three workflows; reuse the hex illustrations).
4. **Plaiq worked example (real, Metrica G3)** → kept; **plus** the illustrative Fused Insight Card (D3), clearly distinguished from the real one.
5. **How Matrix works + Three layers (tabbed)** → **Foundation**: the three tiers (reuse the tabs component) + the Parity callout. The Track A proof table moves here or to `/security`'s data-handling detail.
6. **New: Trust, privacy and data sovereignty** (links to `/security`).
7. **FAQ** → rewritten; the held data-handling and LLM-processing answers wait on the lawyer.
8. **Contact** → pilot form.
- **New static pages:** `/security`, `/privacy`, `/terms`. Each needs a `build.sh` copy-list change and a check of how Cloudflare resolves the paths (e.g. `/security/` vs `security.html`).

## Workstreams

- **A — Align (first):** update `website.md` (done in part by this plan) then re-sync `AGENTS.md`; fix `AGENTS.md`'s hand-written product text; add a v2 decision log (D1–D4) to `content/draft.md`; add `noindex`; create the `v2` branch and confirm preview deploys.
- **B — Copy:** write `content/launch-draft.md` section by section, in the same decision-log format as `draft.md`, **with a claim ledger** — every public claim tied to its evidence (a test, a doc, a real result) or marked *override/illustrative/in development*. Nothing ships without a ledger line.
- **C — Page build:** restructure `index.html` and `css/sections/*` per the map above; no new framework; keep the existing design system and the founder-supplied assets (no redrawing).
- **D — Trust pages and form:** draft `/security` (only controls that exist at launch: RLS, pseudonymised prompts, Vault, purge job — never "GDPR compliant"), `/privacy`, `/terms` (with open-data attributions) for **lawyer review**; update the form and its privacy note.
- **E — Meta and SEO:** title/description/OG per the spec, minus any claim the page can't back; a new OG image is a new asset (the founder-supplied-assets rule means I will ask you to supply or approve it, not generate one). Sitemap and robots at launch.
- **F — Launch QA:** check every ledger line, every link, mobile layout, Lighthouse, and the gates below.

## What can be built now vs. what waits on the product

- **Now:** A–E except real screenshots, the live Sign in link and the "Available" chip.
- **Waits on the app:** real product screenshots, enabling Sign in at `app.pontahq.com`, the Tier 1 "Available" chip, and the final hero visual.

## Launch gates (public announcement, not the friends-only URL)

- **G1** — The app behind `app.pontahq.com` is reachable and Sign in works.
- **G2** — Tier 1 "Available" is demonstrable end-to-end on real data.
- **G3** — Metrica and SkillCorner open-data licences are actually verified (including whether marketing use is permitted) and attributions added to the worked-example section and `/terms`.
- **G4** — `/privacy`, `/terms`, `/security` are lawyer-reviewed; LLM sub-processor terms exist; the `/security` page matches what is implemented.
- **G5** — Every claim-ledger line has a source or an explicit label.
- **G6** — Fusion and illustrative-content check: the hero's fusion wording is re-decided against the validation status at that time (validated Tier 3 data, or revised wording), and every illustrative element is still visibly labelled.

## Residual risks of the decisions taken (stated plainly, owned by the founder)

- **D2:** a present-tense fusion capability claim rests on synthetic validation only. The hero's "legacy tools…" sentence is also a comparative claim about products not researched. If a prospect or a vendor challenges either, there is no evidence on file to answer with.
- **D3:** an illustrative card with realistic numbers can be screenshotted or remembered as a real result. The persistent label mitigates this but does not eliminate it, and any use of the card in a sales conversation should be introduced as illustrative.
- **D4:** a public commitment about data use becomes a contractual expectation; the DPA must say exactly the same thing.
- **Unverified licences (G3)** remain the one item that can force a content change (the Metrica/SkillCorner-derived sections) late in the process.

## Not in scope

The Next.js product app, the API, a hosted sandbox or demo environment, any change to the founder-supplied brand assets, and legal drafting beyond producing page drafts for review.
