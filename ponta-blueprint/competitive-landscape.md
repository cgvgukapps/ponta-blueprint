# Ponta Competitive Landscape

See [README.md](README.md) for baseline assumptions and [risks.md](risks.md) for how this document's findings feed the risk register.

## Why this document exists

Before this pass, the blueprint's biggest unstated assumption was that PLAIQ's core metric — decision-making quality from off-ball positioning and pressure — was differentiated enough to build a company around. That assumption had never been checked. It has now been checked, with real research, and the honest finding is uncomfortable enough to shape everything downstream. A realistic startup does this teardown *before* writing more product code, not after — this document is late by the standard of good practice, and should be treated as the most urgent unblock in the whole blueprint.

## Direct competitor teardown

### StatsBomb IQ (Hudl)

The dominant incumbent at the top of the market. Combines event data, StatsBomb 360 (off-ball freeze-frame positioning), and proprietary advanced metrics — most notably **On-Ball Value (OBV)** and **HOPS** (an aerial-duel rating model) — with scouting shortlists, percentile radars, similarity search across 170+ leagues, and direct video linkage. Used by Premier League and EFL clubs, agents, and senior analysts. Pricing is public knowledge at a rough level: several thousand pounds/year for individual users, six figures for full-club packages.

**Overlap with PLAIQ:** High at the recruitment/scouting workflow level (shortlists, comparisons, percentile benchmarking), lower at the specific metric level — OBV is a value-added/possession-value metric family, not the same construct as Decision Quality's "actual choice vs. best counterfactual option" framing. But a buyer evaluating both products would reasonably see them as substitutes.

### SkillCorner Game Intelligence

This is the finding that matters most. SkillCorner's **Game Intelligence** product, used by 250+ teams and leagues, already ships: off-ball movement volume/type/threat scoring, "Passing Options" and "Pass Difficulty" metrics, a 5-level "Overall Pressure Degree" model that factors in space, time-to-impact, and reception difficulty, and an "Execution Under Pressure" metric that explicitly splits every possession into start/end moments to judge decision quality under duress — benchmarked against league and position averages. Described in their own marketing as scoring "not just what happened, but what could have happened."

**Overlap with PLAIQ: severe, not superficial.** This is conceptually the same product — off-ball positioning value, pressure-adjusted decision quality, passing-option quality — already built, already commercially validated at 250+ clients, backed by SkillCorner's own proprietary tracking data (which most clubs already have to buy from them to use it at all).

**One specific thing that's genuinely unverified, not just "probably the same":** SkillCorner's marketing says Execution Under Pressure judges decisions "not just what happened, but what could have happened," which *sounds* like it could mean intent-based scoring of failed actions — but their methodology isn't public, so this can't be confirmed either way. This is a concrete, answerable question for a trial account or discovery call, not something to assume in either direction.

### Impect (now part of Catapult, partnered with SAP)

Built around **Packing** — quantifying how many opposition players a pass or dribble removes from the game, with a weighted variant (bypassing defenders scores higher than bypassing attackers). Used across 160+ competitions, has its own API, connects to video, and now has SAP/Catapult distribution behind it. Conceptually adjacent (both are "quantify the value of an action beyond raw stats") but a different specific construct than POV/Decision Quality — packing is about lines bypassed, not counterfactual option quality.

**A confirmed, not just plausible, point of difference:** Impect's own definition scores players removed "by a **successful** forward pass or dribble" — the word "successful" is doing real work. A defender who reads the game correctly and cuts out a good pass gets full credit; the passer who made the right decision gets nothing, because Packing has no mechanism for crediting a decision that didn't complete. That's a specific, citable, outcome-only design choice in a named competitor's headline metric — not a guess.

**A newer development worth tracking closely:** the "now part of Catapult" parenthetical above understates how significant this is. Catapult — the dominant wearable/GPS vendor in football — acquired Impect outright (announced October 2025, closing 2026), explicitly to combine its athlete-performance/wearable suite with Impect's tactical scouting data into "a unified intelligence platform." Catapult's own public materials are already using language like "bridging the gap between GPS and tactical reality." Nothing confirms they've shipped a specific decision-quality-vs-fitness-load visualisation yet, but this acquisition means one company now owns both a wearable-data pipeline and a decision-value metric — the two ingredients behind a new PLAIQ idea explored in [roadmap_v2.md](roadmap_v2.md#feature-3-fitness-reconciled-decision-quality-physical-load--decision-timeline). Worth monitoring as part of the Week 1 teardown refresh below, since this is the most time-sensitive competitive signal found anywhere in this review.

### Google DeepMind TacticAI

Not a commercial threat in the conventional sense: it's a research collaboration (Liverpool FC, Palmeiras, the Brazilian federation), not a product a club can buy, with no public pricing or self-serve access. Relevant for a different reason — it signals that the very top of the market (elite clubs with in-house data science departments) already has access to bespoke, cutting-edge modelling that a solo founder cannot realistically out-build. This reinforces rather than changes the earlier B2B SaaS/small-scale positioning: **the top 1% of clubs are not a realistic near-term customer, regardless of competitive gaps.**

## The honest conclusion

PLAIQ's engine is real, well-built, and well-specified — but "off-ball decision-making quality, pressure-adjusted" is not a blue ocean. SkillCorner has already productised nearly this exact concept at commercial scale. Pretending otherwise would be the single most damaging mistake in this entire blueprint. The question is no longer "is this differentiated" in the abstract — it's "which specific wedge, if any, is still open," and that has to be answered with real customer conversations, not more architecture.

## What's still plausibly defensible

Five angles survived this teardown, in rough order of how defensible they seem:

1. **Vendor-agnostic aggregation.** SkillCorner Game Intelligence only runs on SkillCorner's own tracking data. StatsBomb IQ only runs on StatsBomb's own event/360 data. Impect only on Impect's own event data. All three incumbents are vertically integrated by design — their business model depends on selling you their proprietary data feed, so decoupling the analysis layer from the data source is directly against their commercial interest. PLAIQ's BYOD, multi-connector architecture (`architecture.md`) is structurally positioned to do something none of the three incumbents will do: serve a club that has, say, Second Spectrum tracking and wants this style of analysis without also buying a second vendor's full data stack.
2. **Intent-based scoring of failed actions, not just completed ones.** This is a specific, technically deliberate piece of engineering (§7.3), not a marketing claim: when a pass is intercepted, PLAIQ infers the intended receiver from a start-frame Personal-Control-Zone walk, projects the meeting point the ball was headed toward, and scores that pass exactly as if it had connected — closing what the spec itself names as "selection bias" in every outcome-based metric. There's now a concrete, confirmed contrast for at least one named competitor: Impect's Packing only credits a "**successful** forward pass or dribble," so a well-conceived pass that gets cut out scores zero. Whether StatsBomb OBV or SkillCorner's Execution Under Pressure do anything equivalent is genuinely unknown (see the SkillCorner section above). **The public line on pontahq.com is now "Plaiq scores the decision, not just the outcome"** — a capability description, with a worked example, and still with no named-competitor comparison on the page. Discovery calls should still test whether buyers hear that as differentiated from SkillCorner/StatsBomb; putting it on the homepage does not close that question.
3. **Price-tier gap.** StatsBomb IQ's six-figure club packages and presumably-comparable SkillCorner Game Intelligence bundle pricing leave a real gap below them: clubs outside the top divisions, individual scouts/agencies, and federations that can't justify that spend but still want more than free/basic stats. This is the classic incumbent-price-umbrella wedge, not a novel insight, but it's a real one.
4. **Explainability as a stated design principle, not a footnote.** Public descriptions of all three incumbents' advanced metrics ("proprietary AI models," "proprietary Packing® data") market their models as black boxes to be trusted, not audited. PLAIQ's DQ badge pattern (`frontend.md`) — showing the actual counterfactual comparison, confidence flags on inferred receivers — is a genuinely different design stance that could matter to analysts who need to defend a recruitment recommendation to a skeptical board, not just cite a proprietary score.
5. **Persona/niche focus the incumbents under-serve.** All three competitors are built recruitment/coaching-first. The media/content persona in `features.md` (auto-generated match stories, shareable moment cards) isn't their focus. This is a narrower, unproven wedge, but worth testing.

None of these five is fully validated yet — they're hypotheses a founder can now go test, not conclusions. Note that the off-ball-movement-improves-passing-option idea (§5.1's ΔPOV) isn't listed on its own: it's real and well-built, but conceptually it overlaps closely enough with SkillCorner's marketed "off-ball runs" / "Passing Options" metrics that it doesn't clear the bar for a *confirmed* differentiator the way point 2 does — it folds into the same open question about SkillCorner's internal methodology.

## Validation plan (do this before building more product)

A lean, time-boxed research sprint, not an open-ended research program:

1. **Week 1 — structured teardown (mostly done by this document).** Fill remaining gaps: actual SkillCorner Game Intelligence pricing/bundling terms, whether it's sold standalone or only bundled with tracking contracts, and get a first-hand look if at all possible (trial, demo request, or a contact who already has access). Also worth a specific check: what, if anything, Catapult has publicly shipped from the Impect acquisition's wearable-plus-tactical-data integration — see the note above and [roadmap_v2.md](roadmap_v2.md#feature-3-fitness-reconciled-decision-quality-physical-load--decision-timeline).
2. **Week 2 — 8-10 customer discovery calls**, targeted at the *wedge* segments from above, not top-flight clubs: lower-league club analysts, individual scouts/freelancers, agency recruitment staff, smaller federations. Ask what they use today, what it costs them, what's missing, and — critically — show them the actual DQ badge/explainability concept and gauge reaction, rather than pitching an abstract idea. Include one pointed, closed-ended question aimed squarely at wedge #2: *"When your current tool's data shows a pass got intercepted, does it still tell you whether it was a good decision — or does it just log it as a turnover?"* Anyone using Impect can be answered from the public Packing definition already; anyone using SkillCorner or StatsBomb is a genuine unknown this call can resolve.
3. **Decision point.** Three honest outcomes are possible, and the plan should accept any of them:
   - A specific wedge (most likely: BYOD + price gap, aimed at underserved segments) is validated → proceed with the build order in `architecture.md`, narrowed explicitly to that segment.
   - No clear wedge emerges, but the underlying engine/explainability work is valuable as a differentiator inside a *different* product shape (e.g., a scouting-report generation tool, or a media/content tool, rather than a general analytics platform) → revisit `features.md`'s persona prioritisation before revisiting architecture.
   - The market is saturated enough that this specific product isn't fundable as an independent company → the engine and spec remain valuable as a portfolio piece, a consulting capability, or an acquisition target's IP, which is a legitimate outcome, not a failure, for a solo project.

This validation work is cheap (a few weeks, no engineering) relative to the cost of building the full production stack in `architecture.md` against an unvalidated wedge.

## What the AI-native pivot does and does not change

*Added with the one-product, AI-native revision. These are hypotheses to validate, in the same spirit as the five wedges above, not conclusions.*

**What it plausibly adds.**

- **Vendor-agnostic AI.** Every incumbent's analytics and any AI they add run only on their own data (the vertical-integration point in wedge #1 above). A grounded AI layer over a club's *mixed* sources is structurally something none of them is incentivised to build. This reinforces wedge #1 rather than creating a new one.
- **Grounded, cited, parity-honest answers.** Incumbent AI features (and generic LLM chat added to dashboards) are likely to be black-box. A system where every statement is a cited, deterministic event with a stated confidence, and which refuses to answer from Blocked metrics, extends wedge #4 (explainability) into the AI layer. It is also an answer to the real failure mode clubs are wary of: confident, wrong, unexplained output.
- **Zero-click value from a single export.** A brief generated at upload shortens time-to-value for the segments the GTM plan targets — small analyst teams with no data-science headcount — which is a better fit for the price-tier gap (wedge #3) than another dashboard.

**What it does not change, and the open questions.**

- **The SkillCorner overlap is unaffected.** Game Intelligence's metrics remain the closest analogue to Plaiq's. The AI layer sits on top of those metrics; it doesn't make the underlying metric differentiated. Wedge #2 (intent-based scoring of failed actions) still needs the discovery-call test.
- **Incumbents can add AI.** StatsBomb IQ/Hudl, SkillCorner and Catapult/Impect all have the data and distribution to ship natural-language querying on their own data. The defence is cross-vendor scope and grounding discipline, not the presence of an LLM. Check what each has publicly shipped before the first customer conversation and refresh periodically; this is now the most time-sensitive competitive check alongside the Catapult/Impect integration.
- **Tier 3 is a race, as before.** Catapult now owns a wearable pipeline and a decision-value metric; the Tier 3 flagship has the same "race, not moat" status stated in `roadmap_v2.md` Feature 3. Shipping Tier 1 first, and winning real paired data, is the credible response — not asserting a lead.
- **Generic AI is a substitute risk.** A club could paste exports into a general-purpose assistant. The differentiator is trustworthy, structured, quality-scored data feeding the AI (what Matrix and Plaiq produce) — worth stating in discovery calls and testing, not assuming.

**Additions to the validation plan.** Add to the discovery calls: (1) show the Parity Report and a cited brief and measure whether "every claim links to a replay with a confidence" changes how a buyer talks about trust; (2) ask what AI features their current vendors already offer or have announced; (3) ask whether they hold *two sources for the same match* or any *overlap window between an old and new vendor* — the single most valuable validation asset for Tier 2/3; (4) probe who owns wearable data at the club and whether a sports-science champion is reachable.
