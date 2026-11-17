# Ponta Data Tiers — Progressive Intelligence

See [README.md](README.md) for the document index, [Matrix.md](Matrix.md) for the engine that does the work in each tier, and [ai-layer.md](ai-layer.md) for the AI workflows that sit on top. This document defines **what a club gets depending on what data it brings**, and — just as importantly — **what must be true before each tier's headline claim is honest**.

## The principle

Ponta is one application. Every uploaded file, whatever the vendor, is compiled into one canonical, quality-scored store (see [architecture.md](architecture.md#canonical-store-and-ai-layer)). What changes with the data a club brings is **how much the app can say**, not whether it works. The app must never block on the hardest unproven piece (fusing paired sources on real data) in order to deliver value on the easy, proven one (cleaning and analysing a single source).

Three tiers, each unlocking the next. The tier of a match is stored on the match record (`fusion_tier`) and is read by every AI workflow before it speaks — see the parity rules below.

## Tier summary

| | Tier 1 — Single source | Tier 2 — Two unpaired sources | Tier 3 — Paired match sources |
|---|---|---|---|
| **Club brings** | One provider's export (optical, event, or GPS) | Two providers' data that do *not* cover the same match (training GPS + match optical; old vendor history + new vendor) | Same match, two or more independent sources (GPS + optical, ± events) |
| **Matrix does** | **Normalise** — schema, coordinates, units, quality score, drift flags | **Reconcile** — identity matching, timestamp handling, threshold harmonisation; *calibration only where an overlap exists* | **Fuse** — per-frame combination of complementary signals, source attribution |
| **App delivers** | Tactical AI briefs, natural-language querying over decision quality / pressure / pass option value (optical, event) or physical summaries (GPS) | Longitudinal workload and vendor-transition monitoring | Fitness-reconciled decision analysis (association, confidence-banded) |
| **Status of the engine today** | Built and validated (Track A, real vendors; Plaiq spec + tests) | Identity/sync validated on **synthetic** data only; no real pair exists | Fusion validated on **synthetic** data only; no HR/IMU ingestion; no real pair exists |
| **Can ship before a design partner?** | Yes | Partly (see Tier 2 caveat) | **No** — gated on real paired data |

## Tier 1 — Single source

**Inputs.** One provider's export for a match or session. Three sub-cases behave differently, and the product must say which one it is looking at:

- **Optical tracking, both teams** (SkillCorner and Metrica; a Second Spectrum adapter exists but is fixture-tested only, so it is not presented as supported): the full Plaiq engine runs — pressure, personal control zones, pass option value, decision quality, off-ball movement value, failed-pass intent inference. This is the strongest Tier 1 experience.
- **Event data only** (StatsBomb): the degraded, event-only Plaiq mode runs on reconstructed pseudo-continuity (see `features.md` Part 2). Scores are lower-confidence and must be visually distinguished, not shown on the same band as full-tracking scores.
- **GPS only** (STATSports, later Catapult etc.): positions cover **one team only**, so opposition pressure and control zones are **not computable**. This is a physical-load tier, not a tactical one. Tactical AI briefs are Blocked; physical summaries are available. Do not let the product imply otherwise.

**What Matrix does here.** Normalise only: one standard schema, metres, consistent units, a per-frame confidence score, drift/dropout flags. No reconciliation or fusion claim is made. This is the capability proven by Track A on real vendors.

**What the app delivers.** Post-match brief and natural-language querying over the semantic event store ([ai-layer.md](ai-layer.md)). Everything shown is parity-labelled.

## Tier 2 — Two unpaired sources

"Unpaired" means the two sources do not describe the same moment. Two situations are easy to confuse, and they support different claims:

**2a — Overlap exists.** The club ran both systems on the same players at the same time for some window (e.g. old and new GPS worn together for a few sessions, or a trial period). Here a **calibration relationship** between the two can actually be measured: per-metric bias and variance for the overlapping population. Calibrated harmonisation of load metrics (high-speed distance, sprint count, deceleration load) is defensible, with the calibration's sample size and population stated on the result.

**2b — No overlap.** The club switched vendors with no overlap (or compares training GPS with match optical, which measure different things on different days). Here there is nothing to fit a calibration curve to. What Matrix can honestly offer is: matching players across the two histories, aligning definitions and thresholds explicitly, and **flagging the discontinuity**, not erasing it. Claims of "continuous history" must be limited to what the evidence supports.

**Status.** Identity matching and clock synchronisation are validated only on a synthetic second source derived from the same recording. No real two-vendor pair has been tested. A real overlap window is the single most valuable validation asset, and it is a Tier 2 design-partner ask in its own right, not only a Tier 3 one.

**What the app delivers.** Longitudinal workload view with acute:chronic workload ratio (ACWR) spikes, with vendor transitions annotated and calibration confidence shown. Acute:chronic ratios need weeks of per-player history; there is no meaningful ACWR from a single session. Output is framed as a **workload signal for staff review**, not a medical or injury-risk determination (see [legal-compliance.md](legal-compliance.md#ai-and-health-adjacent-data)).

## Tier 3 — Paired match sources

**Inputs.** The same match from two or more independent sources — typically wearable GPS + optical tracking, optionally + event data.

**What Matrix does.** Fuse: align timestamps and identities, then combine the complementary signals per frame (GPS's acceleration precision, optical's team geometry), with source attribution and confidence per frame.

**What the app delivers.** **Fitness-reconciled decision analysis:** the decision-quality timeline shown against the player's accumulated physical load in the same match, with confidence bands. This is the flagship differentiation bet (see [roadmap_v2.md](roadmap_v2.md#feature-3-fitness-reconciled-decision-quality-physical-load--decision-timeline), now promoted into the committed plan).

### What must be true before Tier 3 can ship

Tier 3 is *not* unlocked by wrapping existing code. It requires all of:

1. **A real paired match.** One match recorded by two independent sources. Nothing in the repo today qualifies; the STATSports export has no matching optical source. This is the gate for everything below and for the validation protocol in `Matrix.md` (absolute agreement, rank preservation, threshold agreement, longitudinal continuity, error propagation).
2. **Physiological channel ingestion.** The STATSports adapter currently *ignores* heart rate and the accelerometer/gyroscope channels (`STATSPORTS_ADDENDUM.md`). Any metabolic-power, heart-rate or deceleration-load field in the schema is **new engineering**, including per-player baselines, which require multi-session history. Heart rate is also inconsistently available and, as health data, carries heightened legal requirements.
3. **A statistical design that does not overclaim.** The most relevant research found a weak correlation between GPS-derived load and decision/psychomotor decline, largely explained by stable between-player differences. The output must be confidence-banded and described as an **association**, never as "fatigue caused this decision". It must not claim to separate "tactical error" from "neuromuscular fatigue" as if that were determinable per event.
4. **Legal review of physiological data and of fusing two vendors' data.** See [legal-compliance.md](legal-compliance.md).
5. **A second internal champion.** Wearable data is usually owned by sports science / medical, not the analyst who buys tactical tools (`Matrix.md` Next Steps).

Until these are met, Tier 3 appears in product copy as *in development*, and the AI layer treats any physiological question on a match below Tier 3 as Blocked (see below).

## Analytics Parity Report

The Parity Report is the mechanism that makes progressive intelligence honest. It generalises the SkillCorner "Analytics Parity" panel built in the prototype (Full / Degraded / Blocked per metric) into a **platform-wide, per-match object** generated at ingestion by `matrix_engine/parity.py`, stored on the match record, and read by the UI and by every AI workflow *before* anything is generated.

For each metric family it records a state and a reason:

| State | Meaning |
|---|---|
| **Full** | The data supports the metric as specified. |
| **Degraded** | Computable at lower confidence (e.g. event-only decision quality, or optical at low frame rate / high occlusion). Reason and magnitude stated. |
| **Blocked** | Not computable from what was supplied (e.g. opposition pressure from GPS-only data; deceleration-load analysis without a GPS source). The product says so plainly and does not fabricate or approximate. |

Rules, which are also enforced in the AI layer ([ai-layer.md](ai-layer.md#parity-and-confidence-honesty)):

- Every AI response reads the Parity Report and per-frame confidence first.
- A Degraded or Blocked metric is disclosed in the response, not silently dropped or substituted.
- Event-only decision-quality scores are displayed as a structurally different confidence class, not as the same number with a smaller badge (resolving the open question in `features.md` Part 2).
- The report carries a **coverage figure** alongside each state: the share of events the engine could actually score for this match, with the cause of the gap (for example ball not detected in 26.4% of frames in the SkillCorner golden match and 32.2% in Metrica Game 3, goalkeeper and set-piece events not scored). A metric labelled Full on a match where a third of events are unscoreable would otherwise hide a large hole.
- The Parity Report is also the **onboarding gate**: after upload, the app tells the club what it can and cannot do with this data, and what a second source would unlock. This is the commercial bridge from Tier 1 to Tiers 2 and 3.

## Context packages by data type

*(Added 17 November 2026.)* The context packages in [context-packages.md](context-packages.md) need both teams tracked for most of what they produce. Within Tier 1 the three kinds of single source support very different amounts of context, and the Data check must say so per package. Planned states, to be confirmed by each package's measurements:

| Package | Optical, both teams (Metrica) | Broadcast optical (SkillCorner) | Event-only (StatsBomb) | GPS only (STATSports) |
|---|---|---|---|---|
| P1 Possession and game state | Full when the ball is detected; inferred share stated | Full or Degraded by ball-detected share | Possession and game state from events (observed); no frame-level possession | Blocked (no ball, one team); game state by upload |
| P2 Team structure | Full | Degraded when players are off screen | Blocked (360 freeze-frames give partial snapshots only) | Own team's shape only; opposition Blocked |
| P3 Relative zones | Full | Inherits P2 | Blocked | Blocked |
| P4 Phase of play | Full | Degraded when possession was inferred | Partial, from event sequences | Blocked |
| P5 Relationships | Full | Degraded off screen and when the ball is undetected | Blocked | Blocked |
| P6 Movement types | Full | Degraded while the runner is off screen | Blocked | Own team only, without ball context |
| P7 Space and value | Full | Degraded off screen | Base values as event-only estimates; adjusted Blocked | Blocked |
| P8 Semantic states | Built from whatever above is Full or Degraded, with the states carried into each episode | Same | Episodes from event sequences only | Physical episodes only |

Paired sources (Tier 3) could later fill gaps (for example GPS positions for players off screen in broadcast tracking), but only after fusion is validated on real paired data.

## How this changes the old product map

| Old framing | New framing |
|---|---|
| Matrix Normalise / Reconcile / Fuse as three sold products | The same three layers, now the **engine inside one app**, whose output is determined by the data tier |
| Plaiq as an independent analytics product | Plaiq's decision-quality math is the **spatial engine** that turns Tier 1+ data into semantic events |
| Plaiq "runs on Matrix output or on its own" | The app always runs through Matrix (at minimum Normalise); the prior optional-dependency framing is retired. Plaiq code remains independently testable. |

Public naming of these pieces at launch is an open item in [website.md](website.md#launch-positioning-one-product-ai-native).
