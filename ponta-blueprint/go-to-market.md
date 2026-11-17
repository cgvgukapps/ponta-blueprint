# Ponta Go-to-Market

See [README.md](README.md) for baseline assumptions and [competitive-landscape.md](competitive-landscape.md) for the wedge hypotheses this GTM plan targets.

## The realistic starting point

Football is a closed, relationship-driven, trust-based industry. Clubs do not self-serve sign up for analytics tools off a landing page the way a typical B2B SaaS buyer does — procurement is slow, incumbent vendor relationships (StatsBomb, SkillCorner, Impect are all multi-year, deeply embedded) are sticky, and a solo/bootstrapped founder has no brand recognition to lean on. GTM for this product looks much more like early-stage B2B in a niche professional community than like a typical self-serve SaaS launch.

## Who to target first (not who you'll eventually serve)

Per [competitive-landscape.md](competitive-landscape.md#whats-still-plausibly-defensible), the price-tier gap and BYOD flexibility point at a specific initial segment — **not** the Premier League clubs that are the incumbents' showcase customers:

- **Clubs already using SkillCorner's tracking hardware** — a specific, concrete first target, not just a generic "smaller club" description. SkillCorner's tracking system is comparatively lower-cost than StatsBomb/Second Spectrum's full enterprise packages, which means clubs running on it are disproportionately likely to be exactly the underserved segment this whole GTM plan targets: real tracking data already in hand, but without the budget or in-house data-science headcount to also buy SkillCorner's own Game Intelligence analytics tier (or a StatsBomb IQ contract) on top of it. This is a large part of why `architecture.md`'s connector build order prioritises SkillCorner-via-file-upload in Phase 2 (`roadmap.md`) even though the live "connect your account" SkillCorner integration waits until Phase 4 — it puts this specific, identifiable segment within reach as early as technically possible.
- Clubs outside the top 1-2 divisions in major football markets, and top divisions in smaller football markets, where a six-figure StatsBomb IQ package isn't in budget.
- Individual scouts, freelance analysts, and small recruitment agencies who need professional-grade analysis but buy tools individually, not through a club procurement process.
- National federations and academy programs, especially in markets not already saturated by the big three vendors.
- Media/content buyers (per the persona in `features.md`), an audience segment the incumbents don't focus product design on at all.

The realistic first customers are ones with a fast "yes," not the most prestigious logo.

## Matrix's ICP: clubs experiencing data fragmentation, specifically

The segment list above was written Plaiq-first (underserved by the incumbent price umbrella). Matrix's value proposition is different — vendor-agnostic interoperability — so its best-fit prospects are the ones who *already feel* the pain of multiple, disagreeing tracking systems, not just the ones priced out of incumbent analytics:

- **A club mid-transition between tracking/GPS providers.** The single clearest trigger: budget cycles, contract renewals, and hardware failures all force this periodically, and it creates an acute, dated pain (historical data becomes discontinuous) that Matrix directly solves — see `Matrix.md`'s "Provider Transition" use case. This is also a segment with an obvious reason to have budget *right now*, not a hypothetical future need.
- **A club running different systems for different contexts** — e.g. GPS in training, optical for home matches, a different (often league-mandated) optical feed for away matches. This fragmentation is routine, not an edge case, for many clubs already.
- **A multi-team organisation** (first team + academy + women's team) where different squads plausibly run different systems, and a shared canonical layer across them is more valuable the more teams there are.
- **A federation**, already named in the segment list above for Plaiq — worth calling out again specifically for Matrix, since a federation overseeing multiple clubs/competitions may have exactly the cross-provider interoperability problem Matrix exists to solve, structurally, not just as one customer among many.
- **A performance consultancy or agency working across multiple clubs.** Distinct from the individual-scout segment above: an agency or consultancy dealing with several clubs' data as a matter of course experiences provider heterogeneity *by definition* — a strong candidate for an early, high-conviction design partner precisely because the pain is their daily reality, not an occasional inconvenience.

This runs on the same founder-led motion below, not a separate GTM effort — per `roadmap.md`'s Phase 0, every warm-intro conversation should screen for both a Plaiq wedge and a Matrix design-partner fit at once.

**The public CTA on pontahq.com is this ICP, condensed:** "Using data from more than one provider, or about to switch?" The contact form asks which providers they use. Do not invent a different public ask.

## AI-native positioning and the tiered design-partner ask

*Added with the one-product revision.* Ponta now goes to market as **one AI-native product** whose value grows with the data a club brings ([data-tiers.md](data-tiers.md)), so the design-partner ask is structured as an ascending ladder rather than a single all-or-nothing pitch:

1. **Tier 1 ask — "send us one export."** Any single provider's export (optical, event or GPS). We return a quality-scored, normalised dataset, a **Parity Report** (what the data can and can't support) and, once the product exists, a cited match brief. Low friction, no second source needed, and the fastest route to a reaction. This is the opening move for every conversation.
2. **Tier 2 ask — "do you have an overlap?"** Two systems worn on the same players at the same time for some sessions (typically during a vendor switch). This is the cheapest real data that validates Reconcile; ask it of any club mid-transition, a multi-team organisation, a federation or a performance consultancy (the Matrix ICPs above).
3. **Tier 3 ask — "same match, two sources?"** GPS + optical (± events) for one match. Needs a second internal champion in sports science/medical (usually a different person from the analyst), and triggers the health-data legal review. Pursue only after the first two rungs build trust.

Screen every warm intro for all three at once, as before. **The Parity Report is also the sales bridge**: showing a club exactly what a second source would unlock on *their own* data is a concrete, non-pushy reason to ask for it.

**What to say, and not say, about the AI.** Lead with *grounded and explainable* — every statement links to a replay and a confidence — not with "AI-powered". Incumbents can add chat on their own data; the differentiator to test in discovery calls is whether cross-vendor, cited, parity-honest answers change how a buyer talks about trust ([competitive-landscape.md](competitive-landscape.md#what-the-ai-native-pivot-does-and-does-not-change)). Do not pitch Tier 3 as available, as real-time, or as separating tactical error from fatigue; pitch it as in development and association-only.

## Two wedges, one to lead with (added 15 November 2026)

The pitch and the product ship in a different order. The public call to action ("using data from more than one provider, or about to switch?") and the Matrix ICP above describe a **Tier 2** problem, which has no real overlap data. The hero promises fused physical and tactical analysis, which is **Tier 3**. What ships first is **Tier 1**: one export in, a Parity Report and a cited brief out.

| Wedge | Buyer | What they get first | Evidence status |
|---|---|---|---|
| **A. Single-export brief** | Small analyst team with one provider, priced out of incumbents (for example a SkillCorner or Metrica club) | Parity Report, cited brief, replay, explainable Decision Quality | Engine proven; value of the brief untested |
| **B. Fragmentation and provider transition** | Club with two systems or mid-switch, multi-team organisation, federation, consultancy | Reconcile, workload continuity | Synthetic only; needs an overlap window |

**Do not choose by argument.** Ask both sets of questions in every discovery call and let the answers decide which to lead with. Until then wedge A is what the first product serves, and wedge B stays an early-access conversation. The website's pilot form already captures role, data stack and primary goal; use it as the structured intake for warm introductions (the site is `noindex`, so introductions are the only traffic).

**Add to every discovery call:** the intercepted-pass question and a price anchor; whether the club's vendor licence allows sending exports to a third-party processor (see [legal-compliance.md](legal-compliance.md#vendor-licence-check-before-the-first-real-export)); and who owns any video the club would want linked to a brief.

## The founder-led motion

At solo/bootstrapped scale, this is directly founder-led sales, not a repeatable outbound machine:

1. **Personal network and warm intros first.** In an industry this relationship-driven, a cold outbound email to a club has a near-zero response rate; a warm introduction from someone the target already trusts is the actual unlock. This means identifying who in your existing network (or one hop away) works in or adjacent to football analytics, scouting, or club operations, before building any outbound infrastructure.
2. **A design-partner program, not a sales pipeline.** Recruit 2-3 early users (individual scouts or a lower-league club analyst are realistic first targets) at a steep discount or free, explicitly in exchange for structured feedback, a testimonial, and permission to reference them as a case study. This directly doubles as the customer discovery work in `competitive-landscape.md`'s validation plan — the same conversations that validate the wedge also become the first pipeline.
3. **Content and community credibility, using assets you already have.** The football analytics community (a real, active, technically literate audience on platforms like X/Twitter, in the OptaPro Forum / StatsBomb Conference orbit, and in open-source football-analytics circles around tools like `kloppy` that the prototype already depends on) is exactly the audience most likely to notice and amplify rigorous public work. **Ponta Website is built at [pontahq.com](https://pontahq.com) but not yet public** — it is planned to launch with the product (see [website.md](website.md#launch-positioning-one-product-ai-native)) and will then be that permanent home.
   - Already on the site: Matrix Track A proof (`#proof`, Metrica and SkillCorner open sample data) and a Plaiq worked example (`#plaiq-data`, Metrica Sample Game 3, anonymised).
   - Still not produced, and correctly **not** on the site: the StatsBomb open-data match write-up (e.g. "we scored every player's decision-making in a World Cup final"). That remains the next credibility piece when it exists — do not put an empty "Public work" slot on the page in the meantime.
   - Consider publishing parts of `ANALYTICS_SPECIFICATION.md` itself (or an adapted version) as a technical blog series. Its unusual rigor — documenting failed formulations and why they were rejected, not just the final formulas — is a genuine credibility asset with a skeptical, technically literate buyer audience that's rare to see published openly. (Weigh this against the IP-protection tension `README.md`'s Deferred section flags as unresolved, before publishing anything wholesale.) Public tone on the marketing site itself is "promote, don't qualify" ([website.md](website.md)); a technical blog series can be more caveat-heavy.
4. **Realistic sales-cycle expectations.** Even a motivated small-club analyst champion likely needs internal budget approval; plan runway assuming months, not weeks, from first conversation to signed contract, especially for anything beyond a free/discounted design-partner arrangement.

## Custom tools: a deeper version of the design-partner track, not a separate business

An evolution of the design-partner idea above worth being deliberate about: instead of just recruiting design partners for feedback, go further with a small number of them — build a bespoke tool or dashboard view specifically for that club's stated need, delivered through the same Ponta app they already use. Done right, this is one of the highest-leverage moves available to a solo/bootstrapped founder in this market, for three reasons:

- **It's the fastest way to learn what clubs actually want**, at a level of specificity generic feedback calls never reach — a club asking for a custom view is telling you exactly what data they want extracted and how, not describing a vague pain point.
- **It's a genuine relationship differentiator** against incumbents who sell a fixed product, not a partnership — "Ponta built us something specific" is a much stronger reference story than "Ponta's dashboard is fine."
- **It directly seeds `roadmap_v2.md`'s Team Context ML idea** — deep one-to-one work with a design partner is exactly the kind of engagement that surfaces the "here's specifically what our club values in a decision" signal that idea needs real usage data to learn from.

Two disciplines make this sustainable rather than a trap, both owned by other documents so this section doesn't duplicate them:

- **Build within the shared platform, not as one-off forks** — see `architecture.md`'s per-tenant configurable-module principle. A bespoke tool that can't graduate into the core product with minimal rework defeats the entire point of doing this.
- **Get the IP/generalisation rights settled in the contract before starting, not after** — see `legal-compliance.md`'s new services-contract flag. Without an explicit clause, a club could reasonably assume (or demand) exclusivity over something built specifically for them, which would block exactly the generalisation this track exists to enable.

**Keep it deliberately small.** This is time-for-money work for a solo founder, competing directly against the productised build in `roadmap.md`. The discipline that keeps it from becoming an unplanned consulting business: cap it at the same 2-3 design partners already recommended above, not "custom work for any club that asks."

## Pricing philosophy

*Revised note:* the AI layer adds a usage-driven cost (briefs per match, explorer queries) that flat per-seat pricing can hide; measure it before choosing between capped, metered or bundled Ask Ponta usage ([operations.md](operations.md#llm-costs-and-ai-operations)). The principles below are otherwise unchanged.


Positioned explicitly below the incumbents' price umbrella (StatsBomb IQ's "several thousand pounds/year for individual users, six figures for full club packages" per `competitive-landscape.md`) rather than trying to match feature-for-feature at their price point. Two additional shapes worth planning for once the BYOD vs. platform-licensed distinction in `architecture.md` matters commercially:

- **BYOD tiers** stay pure software margin — price against the value of the analysis layer alone, since the customer already carries their own data cost.
- **A future platform-licensed "data included" tier** (see `architecture.md`'s Future Evolution section) prices differently, with data-licensing cost baked in as COGS — not decided in detail here, since it depends on a commercial negotiation that hasn't happened yet.

## What this document deliberately does not do

It doesn't commit to specific pricing numbers, doesn't name specific target accounts, and doesn't build a CRM/outbound sequence — all of that is premature before the validation calls in `competitive-landscape.md` happen. This document exists to make sure the *shape* of the go-to-market motion is realistic before any of that detail gets filled in.
