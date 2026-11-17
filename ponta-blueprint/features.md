# Ponta Features

See [README.md](README.md) for the baseline assumptions this document builds on, and [architecture.md](architecture.md) for the system these features run on.

> **Revised for the AI-native one-product strategy.** Parts 1–2 below (the prototype audit and the dual-mode finding) remain accurate history and still describe what the engines can do. Parts 3–4 (shared core, persona layers) and the sequencing below are **reorganised around three AI workflows and three data tiers** — see [Part 5 — AI-native workflows and revised sequencing](#part-5--ai-native-workflows-and-revised-sequencing) and [data-tiers.md](data-tiers.md) / [ai-layer.md](ai-layer.md). "PLAIQ" in this document means the decision-quality engine (`plaiq_engine`) inside the Ponta app. Persona features in Part 4 are retained as *use cases served by the same workflows*, not as separate product lines.

This document was rewritten after a comprehensive re-audit of the prototype (every mode in `app.py`, every module in `analytics/`, `ingestion/`, `statsbomb/`, and the auxiliary `roster/`/`replay.py`/`scripts/` code) — not just the parts covered in the first pass. The goal is to separate three things that were previously blurred together: **what the prototype demonstrably does today**, **what that implies is buildable at low risk**, and **what would be genuinely new product work**.

## Part 1 — What the prototype actually does today

### Data sources and operating modes

The prototype supports **six data providers across two fundamentally different operating modes** — this is the single most important correction to the earlier version of this document, which implied one uniform tracking-based pipeline.

| Mode | Providers | Spatial input | What it can compute |
|---|---|---|---|
| **Full-tracking** | Metrica Sports, FIFA EPTS, Second Spectrum/Genius, SkillCorner | Continuous player/ball positions + velocities, every frame | The complete engine: live kinematic Personal Control Zones, Pass/Carry Decision Quality scored against real-time alternatives, off-ball Movement Value/ΔPOV timelines |
| **Event-only** | StatsBomb (open data) | No continuous tracking at all — reconstructed from event sequencing + optional 360° freeze-frames | A *degraded but real* version of the same engine: freeze-frames are stitched into pseudo-continuity (`statsbomb/stitch.py`), anonymous freeze-frame dots get named via a 5-tier identity cascade (`statsbomb/identity.py`), and Pass DQ / a simplified ring-based Carry DQ run on zero-velocity Personal Control Zones (`statsbomb/sb_dq.py`, `statsbomb/sb_pov_adapter.py`) |

This dual-mode reality is a bigger deal than it first looks — see [Part 2](#part-2--the-dual-mode-finding-and-what-it-changes) below.

**What's wired into the live app UI today** vs. what only exists in the offline `preprocess.py` pipeline:

| Provider | In `app.py`'s UI | Match selection | Notes |
|---|---|---|---|
| Metrica / FIFA EPTS | Yes (Analysis + Player Performance modes) | **None — hardcoded to one fixed match** (Game 3) | Games 1–2 exist as processed files but aren't selectable in the UI |
| SkillCorner | Yes (Analysis + Player Performance modes) | Dropdown over a local `matches.json` | 10 fps broadcast tracking; ships with its own in-app "Analytics Parity" disclosure (see below) |
| StatsBomb | Yes (Event Explorer, Match Flow, Player Performance modes) | Competition → season → match dropdowns, live via `statsbombpy` | Only provider with a real, dynamic match browser today |
| Second Spectrum / Genius Sports | **Offline pipeline only** (`preprocess.py --provider secondspectrum`) | N/A | Fully implemented ingestion adapter, but no path to load it from the running app |

### The full engine — every named metric

| Metric | What it measures |
|---|---|
| Hexagonal grid | Tessellates the pitch into ~1 m² zones so every other metric is zone-native |
| 1v1 Pressure | How close the nearest opponent is (1–10 scale) |
| Zonal Pressure | Mean pressure from the 3 nearest opponents — a "triangle of coverage" |
| Personal Control Zone (PCZ) | Who can reach each zone before the ball, from real player kinematics |
| Static Expected Threat (xT) | How dangerous a zone is to occupy, independent of who's there |
| Hex Value / Effective Value | xT × team control × (1 − pressure discount) — "how valuable is this space right now" |
| Event ΔV | How much the ball's position value changed during an event |
| Movement Value | Value of where a player arrived off the ball, and how much of that was earned by the run |
| Pass Option Value (POV) | Quality of a teammate as a passing option, factoring in threat, control, interception risk, and distance |
| ΔPOV | Did a player's off-ball run make them a *better* passing option? |
| Pass Decision Quality | Actual receiver's POV vs. the best option available at the moment of the pass |
| Carry Decision Quality | Was the direction/distance of a dribble the best of the kinematically reachable alternatives? |
| Failed Pass Decision Quality | Scores *intercepted* passes by inferring the intended receiver from trajectory + PCZ geometry — closes the "only completed passes get scored" bias present in most competing metrics (see [competitive-landscape.md](competitive-landscape.md#whats-still-plausibly-defensible)) |
| DQ Rating | One number per player per match: the mean of all their scored decisions |
| Dispossessed-carry flag | Visual/contextual flag (not a separate score) when a carry ends in a turnover |
| Possession inference | Frame-level "who has the ball" used to tag applied vs. received pressure |
| Hex occupancy / density | Team presence per zone over time — a code-level capability not yet in the design spec's table of contents |

Full formulas and design rationale live in `ANALYTICS_SPECIFICATION.md`; this table is the product-facing summary.

**A note for engineering rigor before this becomes the audited v1 baseline** (belongs in [engine-governance.md](engine-governance.md), flagged here so it isn't lost): the re-audit found a handful of small code/spec mismatches worth a cleanup pass — a ring-distance formula that doesn't match the spec's written version, inconsistent kinematic defaults (`v_max`/`reaction_time`) between a couple of functions, a few dead/unused constants, and outdated prose describing an old version of the off-ball trend line's smoothing window. None of these are large, but they should be resolved and locked down before `engine_version` numbering (per `engine-governance.md`) starts being treated as authoritative.

### The app today — six modes, no export/video/comparison

| Mode | Available under | What it does |
|---|---|---|
| Analysis | Metrica, SkillCorner | Frame-by-frame pitch scrubber with toggleable overlays (PCZ control zones, value-surface heatmap, pass corridors, carry-alternative heatmap, failed-pass intent geometry) and a live per-frame player analytics table |
| Player Performance | Metrica, SkillCorner | Single-player identity card + the Decision-Quality timeline and off-ball Movement Value/ΔPOV timeline, both with trend-line overlays |
| Event Explorer | StatsBomb | Step through individual events with a 360° freeze-frame pitch view and per-event DQ badges |
| Match Flow | StatsBomb | Multi-event arrow map over a chosen time window |
| Player Performance (StatsBomb variant) | StatsBomb | Lighter version of the tracking Player Performance mode (Event ΔV, Carry Movement Value, DQ timeline where identity/360 allow) |

**Explicit gaps confirmed by this audit** — these are not partially-built features to "productise," they are genuinely new work:

- **No file upload anywhere in the running app.** Second Spectrum/Genius data can only be loaded via the offline CLI pipeline.
- **No export or download of anything** — no chart export, no CSV export, no report/PDF generation. Every persona feature in Part 3 that involves a report or shareable asset is new work, not a UI wrapper on an existing capability.
- **No video integration**, despite a `.video_cache/` folder with a handful of `.mp4` files sitting in the repo — nothing in the codebase reads or writes it. It's an orphaned artifact from an earlier experiment, not a built capability.
- **No player search, and no multi-player/side-by-side comparison** — Player Performance is strictly one player at a time, selected from a dropdown.
- **No cross-match player identity** — see the design constraint below, unchanged from the previous version of this document.

### An existing explainability pattern worth deliberately keeping

The SkillCorner sidebar has an "Analytics Parity" info panel that plainly tells the user which specific metrics are **Full / Degraded / Blocked** for that data source (e.g., no xG-based terminal value without shot data). This is a real, working precedent — not just a plan — for the "explainability as a design principle" wedge already identified in [competitive-landscape.md](competitive-landscape.md#whats-still-plausibly-defensible): PLAIQ already has the instinct to tell users what it can't confidently compute, rather than silently degrading. The production UI should generalise this pattern across every provider (see [Part 2](#part-2--the-dual-mode-finding-and-what-it-changes)), not just SkillCorner.

Confidence is also already surfaced at the individual-inference level, not just the provider level: the failed-pass intent inference carries a strong/medium/weak confidence flag, and StatsBomb's freeze-frame player-identity resolution carries its own per-player confidence tier. Both should be treated as instances of one general "confidence-aware scoring" UI pattern, not one-off features.

## Part 2 — The dual-mode finding, and what it changes

The event-only StatsBomb path isn't a lesser fallback bolted onto the "real" engine — it's a second, deliberately engineered mode that reuses the same core math (`analytics/pass_option_value.py`, `analytics/pitch_control.py`) through a StatsBomb-specific adapter layer, rather than a fork of the model. That has real product implications worth carrying into other documents:

- **A cheaper, tracking-free entry tier becomes plausible.** Most of the target segments identified in [go-to-market.md](go-to-market.md) (lower-league analysts, individual scouts, agencies) are far more likely to already have event data (StatsBomb, Wyscout, Opta) than an expensive tracking-data contract. An "Event-Only" or "Lite" analytics tier — same DQ/POV concepts, explicitly lower-confidence, badge-flagged as such — could be a real acquisition wedge below the full-tracking product, not just a StatsBomb-specific curiosity.
- **It reinforces, rather than competes with, vendor-agnostic aggregation** (wedge #1 in `competitive-landscape.md`): the platform's value proposition becomes "we compute Decision Quality on whatever spatial data you already have, tracking or not," which is a strictly broader pitch than "we need your tracking-data vendor's feed."
- **It changes the shape of the confidence-tier UI work.** The "Full / Degraded / Blocked" pattern already built for SkillCorner needs to become a first-class, provider-agnostic product concept (e.g., a persistent "Analytics Fidelity" indicator per match), not a one-off info box — this belongs in [frontend.md](frontend.md)'s design principles and should be added there.
- **It's a genuine engineering-governance question, not just a UI one**: should Event-Only DQ scores be shown in the same numeric scale/band as Full-Tracking DQ scores (risking false equivalence), or visually distinguished as a structurally different confidence class? This is worth resolving deliberately in [engine-governance.md](engine-governance.md) rather than defaulting to "same number, smaller badge."

None of this is scoped or committed yet — it's a new strategic option this audit surfaced, flagged here so it can be picked up explicitly in `go-to-market.md`, `frontend.md`, and `engine-governance.md` rather than silently forgotten.

## Design constraint that shapes everything below

There is one real data-model gap between "single-match analytics prototype" and "multi-persona product": **the prototype has no cross-match player identity.** `roster/build_roster.py` resolves who's who *within one match* (jersey-slot vs. Player-ID vs. substitutions), but there is no concept of "this is the same human across 20 matches." Almost every persona feature below that isn't purely single-match (comparisons, trends, shortlists, season ratings) depends on solving this first. It's tractable — StatsBomb already ships stable `player_id`s across matches in a competition; tracking-only providers (Metrica/SkillCorner/Second Spectrum) will need name-based reconciliation or a manual "confirm this is the same player" step — but it should be treated as core infrastructure, not a persona-specific feature, and built early. (Note: this is reconciling identity *within one provider's data* across matches — reconciling the same player across *two different providers* is a separate, harder problem, explicitly scoped out of v1 in [engine-governance.md](engine-governance.md#cross-provider-player-identity-an-explicit-non-goal-for-v1).)

**Before investing further in this feature set: see [competitive-landscape.md](competitive-landscape.md).** A competitive teardown found significant overlap between PLAIQ's core engine and SkillCorner's existing Game Intelligence product — the persona/feature prioritisation below should be revisited once the validation plan in that document produces real customer feedback, since it may sharpen which wedge (if any) these features should be built around first.

## Part 3 — Shared platform core (every persona depends on this)

Revised from the previous version to be explicit about what's a UI wrapper on existing capability vs. genuinely new work, per Part 1's findings:

- **Match Library** — connect a data source or upload files, see ingestion/processing status, browse matches by competition/team/date. **This is new work, not a productisation of an existing screen** — the prototype has no upload UI and no dynamic match browser for Metrica or Second Spectrum. Per the three connector types in [architecture.md](architecture.md#3-data-ingestion-connectors), a StatsBomb or SkillCorner customer gets a one-time "Connect your account" flow and matches sync automatically, while a Second Spectrum/Genius customer (and most Metrica customers) will use the upload flow every time, since no self-serve API exists for them. The Match Library UI needs to make this distinction obvious per provider up front.
- **Match Replay & Event Explorer** — the pitch view: scrub frames or step through events, toggle overlays (pressure, PCZ control zones, value-surface heatmap, pass corridors, carry alternatives). This one genuinely does descend directly from `app.py`'s Event Explorer / Analysis mode — the most mature, reusable UI in the prototype.
- **Player Profile** — identity, physical profile, on/off-pitch interval, spells across substitutions, and (once cross-match identity exists) a season-level view.
- **Player Performance Timeline** — the Decision-Quality timeline + off-ball Movement Value timeline, with the trend-line overlays already designed in `ANALYTICS_SPECIFICATION.md` §5.1/§7.4. Also directly descended from the prototype.
- **Decision-quality explainability** — the existing DQ badge pattern (`POV(actual) ÷ POV(best available)`, with confidence flags for both inferred failed-pass receivers and — per Part 1's finding — StatsBomb's freeze-frame identity resolution) surfaced everywhere a score appears, not just in the event inspector. This is a trust feature, not a nice-to-have: coaches and scouts are structurally skeptical of black-box ratings, and "show your work" is a real differentiator against competitors who just publish a number.
- **Analytics Fidelity indicator** *(new, per Part 2)* — a persistent, per-match, provider-agnostic version of the SkillCorner "Analytics Parity" panel, telling the user plainly whether they're looking at Full-Tracking or Event-Only analytics and which specific metrics are affected. This generalises something the prototype already does for one provider into a platform-wide trust feature.
- **Org/workspace management** — seats, roles, data-source connections, billing (billing itself deferred, but the seams for it belong here).

## Part 4 — Persona layers on top of the shared core

### Recruitment / scouting

- Shortlists/watchlists with scouting notes attached to a player.
- Player comparison — side-by-side DQ Rating, POV tendencies, pressure-adjusted performance, across two or more players. **New work**: the prototype has no comparison view at all today (confirmed in Part 1), and this also requires cross-match identity for anything beyond one match.
- Style/role fit filtering — e.g. "high DQ specifically under high zonal pressure" as a proxy for composure, or POV-under-pressure tendencies as a proxy for a player's decision-making archetype.
- Exportable scouting report (PDF/shareable link) — headline IQ numbers plus the supporting evidence (key clips of Optimal/Poor decisions), generated from data already computed at ingestion. **New work**: confirmed there is no export capability of any kind in the prototype today.

### Coaching / performance analysts

- Team-level dashboards — average DQ by position/phase of play, which is a straightforward aggregation over the existing per-event Decision Quality output.
- Opposition analysis — where an upcoming opponent's hex-value/pressure surfaces show they concede space, built from the same `hex_value` model already used for the home match view, applied across an opponent's recent matches.
- "Coachable moments" — an auto-surfaced feed of a team's or player's Poor/Suboptimal-banded events (§7.1/§7.2 thresholds in `ANALYTICS_SPECIFICATION.md` already do this classification), essentially a pre-filtered highlight list for a training-ground review session.

### Media / content

- "Moment" export — a single event rendered as a shareable image/clip with its value/POV overlay baked in, for social or broadcast graphics. Data exists per event; the rendering-to-shareable-asset pipeline and video integration are both **entirely new work** — the prototype has no export and no wired-up video capability (see the `.video_cache` finding in Part 1).
- Auto-generated match story — "Top 5 Optimal decisions" / "Costliest Suboptimal decisions" of a match, ranked by the existing DQ score × Event ΔV, no new modelling required (though still new UI/generation work).
- Pre-match briefing — a generated one-pager of a player's or team's historical tendencies for pundits/commentators to reference.
- **Event-Only tier is a natural fit here too** *(new, per Part 2)* — media/content customers are especially unlikely to have a tracking-data relationship, making the StatsBomb-style event-only mode a plausible way to serve this persona without requiring an expensive data prerequisite.

*(Video-linked features like clip export are new work — see the Deferred section in [README.md](README.md) for why.)*

## Part 5 — AI-native workflows and revised sequencing

### The shared core, revised

Part 3's shared platform core is retained, with these changes. **New core capabilities:** the **Parity Report** as a per-match, platform-wide object (generalising the Analytics Fidelity indicator in Part 3 — same intent, now computed by `matrix_engine/parity.py` and consumed by the AI), the **semantic event store**, and **Ask Ponta**. **Match Replay & Event Explorer** and **Player Performance Timeline** remain, now fed from `trajectories.parquet` / `semantic_events.parquet`. **Decision-quality explainability** extends to AI output: every AI statement shows its cited events and confidence.

### The three AI workflows (full spec in [ai-layer.md](ai-layer.md))

1. **Post-match / session brief** — zero-click one-page brief generated at ingestion, with clickable pitch-replay citations and confidence badges. Serves coaching, recruitment and media from one artifact (Part 4's "coachable moments", "auto-generated match story" and "pre-match briefing" ideas become configurable views of it).
2. **Ask Ponta (natural-language explorer)** — typed-filter queries over the semantic store, returning a summary plus a replay playlist. Replaces the Part 4 filtering/shortlist-search ideas as the primary query surface.
3. **Data-quality and parity gate** — onboarding assistant that explains what the uploaded data can and cannot support and what a second source would unlock.

A fourth, **cross-provider workload and transition monitor**, follows the first real Tier 2 data.

### Features by data tier

- **Tier 1 (ships first):** all of the above over single-source data. Optical tracking gives the full tactical set (decision quality, pressure, control zones, failed-pass inference); event-only data gives the degraded, separately-labelled set; GPS-only data gives physical summaries only (opposition pressure and control zones are Blocked — positions cover one team).
- **Tier 2:** longitudinal load and vendor-transition monitoring. Only with an overlap window can calibration be claimed; otherwise the transition is annotated, not smoothed.
- **Tier 3 (gated on real paired data, heart-rate/IMU ingestion, baselines and legal review):** **fitness-reconciled decision analysis** — decision quality shown against accumulated load in the same match, confidence-banded and framed as an association. It must not claim to distinguish tactical error from fatigue per event, and must not make injury-risk statements. See [data-tiers.md](data-tiers.md#what-must-be-true-before-tier-3-can-ship) and [roadmap_v2.md](roadmap_v2.md#feature-3-fitness-reconciled-decision-quality-physical-load--decision-timeline).

### Still-open gaps (unchanged by the pivot)

No export/report/PDF capability, no video, no player search or comparison, no cross-match player identity, and no file upload exist in the prototype; all remain new work. The AI brief gives a first shareable artifact (a rendered page), which partially addresses the export gap, but a PDF/shareable-link export is still new work.

### Revised sequencing (Now / Next / Later)

- **Now:** extraction; Parity Report; semantic builder; Tier 1 ingestion (optical + event-only + GPS-physical); post-match brief; Ask Ponta; parity gate; web app.
- **Next:** cross-match player identity (unlocks trends/comparisons and is also needed for per-player baselines), report export, StatsBomb pull connector, Tier 2 workload monitor once real overlap data exists.
- **Later (gated):** Tier 3 fitness-reconciled analysis; opposition analysis; media moment export (needs video/export capability); youth/academy support; platform-licensed data tier.

## Recommended sequencing (Now / Next / Later) — *pre-pivot, superseded by the revised sequencing above*

Given solo/bootstrapped + small-scale-first from [architecture.md](architecture.md), the honest sequencing is:

- **Now (v1 MVP):** a short engine cleanup pass on the code/spec mismatches noted in Part 1 (cheap, and it's the last chance to fix them before `engine_version` numbering treats the current behaviour as canonical); then the shared platform core, single-match scope, one persona's shortlist/report feature shipped shallow-but-real to prove willingness to pay — most likely recruitment/scouting, since it has the clearest "would pay for this" buyer (a scout report is a familiar deliverable).
- **Next:** cross-match player identity (unlocks comparisons/trends), then the rest of the scouting layer, then team dashboards for coaching. Evaluate the Event-Only entry tier here too — it's cheap to expose once the StatsBomb adapter is already productised as part of the core connector work, and it may be a faster path to paying customers than waiting on tracking-data relationships.
- **Later:** media/content layer and opposition analysis — valuable, but the most dependent on volume of ingested matches to be useful, so naturally later once the data pipeline is proven.

## Future evolution: platform-licensed data as a second market

Everything above assumes bring-your-own-data: the customer already has a vendor relationship, and Ponta only adds the analysis layer. That's the right way to start (zero data-cost exposure, fastest path to a working v1), but it structurally excludes anyone who *doesn't* already pay for StatsBomb/SkillCorner/Metrica/Second Spectrum — which is most smaller clubs, individual scouts, agencies, and media buyers who'd never justify an enterprise data contract on their own.

If Ponta licenses data directly from a provider at the platform level (a business-development effort — see [README.md](README.md)'s deferred list), a new tier becomes possible: **"Ponta Data included"** — the customer just picks a competition and starts analysing, no vendor relationship required. This doesn't replace BYOD, it sits alongside it as a second acquisition channel aimed at exactly the segment BYOD can't reach, and it changes the revenue model for that tier from pure software margin to a data-cost-plus-markup model (the licensing fee becomes COGS baked into the subscription price).

This is explicitly **not** v1 scope, and shouldn't be — it requires its own commercial negotiation with a provider before it's buildable at all. The one thing worth doing now: `architecture.md`'s data model already anticipates this (matches tagged by ownership model — customer-connected vs. platform-licensed) precisely so this tier can be added later without a schema migration.

## Explicitly deferred (fold into README.md's list)

- Which specific provider(s) to approach for a platform-level licence, pricing/margin structure for a data-included tier, and how it's positioned against the BYOD tiers — a pricing/GTM conversation for later, not decided here.
- Whether the Event-Only tier (Part 2) gets its own separate pricing, or is simply what a BYOD customer sees automatically when they connect an event-only data source — a pricing/packaging decision, not a technical one, and one `go-to-market.md` hasn't addressed yet.
- The engine cleanup items noted in Part 1 (ring-distance formula, kinematic default inconsistencies, dead constants, outdated trend-line prose) are noted here for visibility but belong to `engine-governance.md` to actually resolve and track.
