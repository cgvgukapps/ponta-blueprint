# Plaiq review: effectiveness and proposed improvements

> **Status: proposal, 18 November 2026.** A review of Plaiq's models (pressure, control, threat, pass option value, decision quality, failed-pass inference) in the light of what the reference data and FIFA's published approach showed this week. Nothing here changes the engine; each proposal is a measured, versioned change for the founder to approve. Sources: `ANALYTICS_SPECIFICATION.md` (frozen prototype), `CONTEXT_STRUCTURE_REPORT.md` (platform), [fifa-alignment.md](fifa-alignment.md), [engine-governance.md](engine-governance.md).

## The short answer

- **Plaiq's maths is specified and tested; its football validity is not.** "Validated" in the README means the code matches the specification. No Plaiq output has yet been compared with an independent reference or an expert, and [engine-governance.md](engine-governance.md#construct-validity-and-reliability-of-decision-quality-added-15-november-2026) already lists the open questions: the decision-quality bands were tuned on the match used for the worked example, and the passer is the best option in about 65% of scored passes.
- **This week gave Plaiq its first external evidence, and two of its building blocks did not do the job the context layer asked of them:**
  - the **failed-pass inference** did not help line breaks (κ 0.27 to 0.26). *(Corrected 18 November 2026: the first reading, that it finds the nearest receiver rather than the target, was not established. On SkillCorner's data a failed pass's end point is already SkillCorner's own target, so Plaiq's inference cannot be tested there; see step 1 below.)*
  - **pass option value** ranks a pass's worth, but is not a test of whether a team-mate is available. As a filter for line-breaking options it reached κ 0.11.
- **The references to test the rest now exist:**
  - SkillCorner's per-pass labels: expected pass completion, threat, the targeted player (including for failed passes), the list of passing options;
  - the German league's per-shot pressure and expected goals, and its 25 Hz tracking;
  - Wyscout's 1,941 matches of events;
  - the founder's blind replay method.
- **FIFA's approach shows where Plaiq is thin:**
  - its pressure uses distance, the angle of the presser and the number of pressers, where Plaiq uses the nearest opponent's distance alone;
  - its value of a pass rests on breaking the opposition's units, which Plaiq does not see;
  - its expected goals is learned from historical shots, where Plaiq's threat surface is a hand-built geometric formula.

## Step 1 results: the reference harness (18 November 2026)

`tools/plaiq_references.py` and `PLAIQ_REFERENCES_REPORT.md` in `ponta-platform`. Measurement only, default player profiles.

| Model | Reference | Result |
|---|---|---|
| Completion (control × no interception) | 12,103 SkillCorner passes completed or not | AUC **0.78**; SkillCorner's own model 0.90; rank correlation with it 0.62 |
| Pass option value (targeted player) | SkillCorner's threat; its "dangerous" flag | Rank correlation **0.28**; AUC **0.53** (close to chance) |
| Pressure on the shooter | The league's shot pressure (164 shots) | 1v1 **0.30**; zonal (three nearest) **0.49** |
| Threat surface | The league's xG; Wyscout zone goal rate of shots; Wyscout scoring within 10 s | 0.54 (goals AUC 0.67 against xG's 0.81); **0.81**; 0.51 |
| Failed-pass target | — | Not measurable here: SkillCorner's failed-pass end point is its own target, and the other feeds name no intended receiver |

**What this changes in the programme:**
- **Pass option value is the weakest model,** so splitting availability from value and rebuilding the value part moves up.
- **Pressure version 2 now has a reference to beat:** count the pressers.
- **A data-driven threat surface** has two Wyscout baselines to beat.
- **Failed-pass targets need a judged reference** (the replay method) before any change.

## Step 3 results: availability and value split (18 November 2026)

`plaiq_engine/analytics/option_split.py` (0.1.0) writes POV as availability × value, exactly, as a new metric beside POV; POV and decision quality are untouched. Measured on SkillCorner's 20 matches (fit / held out):

| | Result |
|---|---|
| Availability predicting completion | AUC **0.79 / 0.77** (POV 0.66 / 0.66) |
| Value of the targeted player against SkillCorner's threat | **0.79 / 0.78** (POV 0.30 / 0.26) |
| Value against SkillCorner's "dangerous" flag | AUC **0.87 / 0.88** (POV 0.54 / 0.51) |
| Availability picking SkillCorner's listed passing options | AUC 0.67 / 0.66 |
| Line-breaking options from availability (threshold chosen on fit) | Ponta about 5 a pass, SkillCorner about 0.5; κ 0.01 |

- **Pass option value's weak scores came from multiplying two sound halves,** not from either half. The split should be how Ponta reports a passing option: how likely it is to arrive, and what it is worth if it does.
- **Decision quality still uses POV,** so changing it is the decision-quality study's question, not this step's.
- **Passing options are offers to receive** (FIFA's term), which is movement. That moves line-breaking options to P6.

## Step 4 results: pressure version 2 (18 November 2026)

Directional, count-aware pressure (`analytics/pressure_v2.py` 0.1.0, after FIFA's EFI inputs and Andrienko et al. 2017) was measured against SkillCorner's pressure level on 33,270 moments and the league's shot pressure.
- **Not adopted.** Plaiq's existing measures win on both references: 1v1 against SkillCorner (rank correlation 0.82, weighted κ 0.80 held out), zonal against the league (0.49).
- **The two references, like FIFA's two notions, measure different things:** the nearest engaging defender (coded direct, indirect or no pressure), and collective pressure on the ball.
- **Correction:** step 1's reading that 1v1 pressure is weak came from the wrong reference.
- **Proposed named bands on the 1v1 scale,** anchored to SkillCorner's levels: 1–2 none, 3–5 low, 6–7 medium, **8–9 high**, 10 very high. For FIFA's three levels: direct ≈ 8 or more, indirect ≈ 3 to 7, none ≈ 1 to 2, to confirm on a judged sample.

## Step 5 results: a learned threat surface (18 November 2026)

Expected threat learned from 970 Wyscout matches (`analytics/xt_learned.py` 0.1.0, 16 × 12 zones) against Plaiq's geometric surface, on data it never saw:
- **On Wyscout's held-out matches it wins:** scoring within 10 s, AUC 0.671 against 0.641.
- **Everywhere else it loses slightly:** SkillCorner's threat 0.87 against 0.89, shot goal rate by zone 0.83 against 0.89, the league's goals AUC 0.62 against 0.67.
- **Not adopted; the geometric surface stays.**
- **The value gap in POV lies in its progressive factor and distance decay** (threat at the receiver alone 0.88 against POV's value part 0.78), so a value change should look there.

## Decision quality grounded, not judged (18 November 2026)

**The founder decided a judge's rating should not be the truth.** Decision quality is grounded three other ways on 12,012 SkillCorner passes: independent maths (SkillCorner's expected value of every option), what happened next, and context. The replay page becomes a review tool for disagreements, with the maths shown.

**Findings:**
- **Failed passes:** Plaiq rates **89% of failed passes optimal**. The failed-pass formula never charges for the failure. Not to be reported as it stands.
- **Completed passes have outcome evidence:** the share of possessions that lead to a shot rises from 1.2% (poor) to 9.1% (optimal); AUC 0.60, on par with SkillCorner's own analogue (0.62). Higher scores go with lower completion: the score rewards ambition, which should be stated.
- **Independent maths agrees weakly** (rank correlation 0.31): Plaiq calls 59% optimal, while SkillCorner's model says the best option was chosen on 37%. The two models define "best" differently: threat against arrival.
- **Context is sensible:** under heavy pressure both models judge choices more kindly.

**Next (proposed):** a decision quality version 2 that scores the choice at the pass as expected value (the option split, availability × value). It would cover completed and failed passes alike, and be measured on the same three groundings with fit and held-out matches. Version 1 stays the default until it is measured.

## Decision quality version 2, measured (19 November 2026)

Expected value at the moment of the pass (`analytics/decision_quality_v2.py` 0.1.0), one formula for completed and failed passes, with the outcome never an input. On 12,112 SkillCorner passes:
- **The failed-pass leniency goes:** optimal 19% of failed passes against version 1's 89%.
- **But it becomes a measure of risk:** it predicts completion with AUC 0.69.
- **It loses the link to chances and to SkillCorner's analogue:** shot AUC 0.51 against 0.60; rank correlation 0.15 against 0.31.
- **Not adopted; version 1 stays.** Version 1's end-of-pass credit (playing the ball into a run) carries the signal.
- **Plaiq's slow ball-speed model makes long forward passes look riskier than they are,** so calibrating it (step 6) comes before any risk-weighted decision quality. *(Corrected 19 November 2026: step 6 measured v1's ball as 30% to 40% too slow on passes up to 30 m and close to right beyond 45 m, so the bias falls on short and medium passes, not long ones.)*
- **Failed-pass decision quality stays unreported** until fixed.

## Step 6 results: ball flight, players and reaction time calibrated (19 November 2026)

`analytics/ball_flight.py` (0.1.0) makes targeted pitch control's physics explicit parameters; with its defaults it equals v1 exactly. `tools/plaiq_calibration.py` measures them on the seven IDSSE matches (25 Hz tracking with ball height; the league's intended recipient for each pass), fitting on four matches and testing on three, then checks SkillCorner's 20.

| | v1 | Measured |
|---|---|---|
| Ball speed (2,685 received passes) | 5 m/s + 0.2 m/s per metre; on average 0.37 s late, median error 0.42 s | **8.5 m/s + 0.165 m/s per metre**; median error **0.20 s** held out |
| The ball along a pass | Each point timed as its own pass (early points slow) | Slows steadily: 33%, 60%, 81% of the length at a quarter, half and three-quarters of the time |
| Player top speed, peak acceleration (204 player-matches) | 9.0 m/s, 5.5 m/s² | Median **8.4 m/s, 4.7 m/s²** |
| Completion AUC, held-out IDSSE matches | 0.719 | 0.739 (measured ball, reaction 0 s) |
| Completion AUC, SkillCorner | 0.781 | 0.793 (SkillCorner's own model 0.896) |

**Not adopted; v1 stays.** What the step shows:
- **The ball-speed model was wrong, and is now measured** from physics alone. Real passes are 30% to 40% faster than v1 assumes up to 30 m.
- **Correcting the physics alone does not improve completion ranking.** The faster ball at v1's 0.7 s reaction scores 0.715 against 0.719, and the measured slowing and player speeds both lower it.
- **The best fit puts reaction time at 0 s,** which is not human. The parameter is absorbing errors in the availability formula, so it cannot be calibrated as a physical quantity in this formula.
- **The gap to SkillCorner's model is structural, not in the constants:**
  - interception counts any point a defender reaches first, with no ball height, reach or time to control;
  - control at the receiver is a fixed sigmoid;
  - neither is a probability (passes rated below 0.1 are still completed about 70% of the time).

**Next (proposed):** a probabilistic, height-aware interception and control model. It would follow the published pitch-control method (Spearman 2018) and use the measured ball flight and player speeds, with reaction and control time fitted only inside it. It would be a candidate beside v1, measured on the same split.

## A probabilistic pass model, measured (19 November 2026)

`analytics/pass_probability.py` (0.1.0), after Spearman (2018), is a candidate beside targeted pitch control. It models:
- uncertain arrival for every player;
- interception in flight along the measured ball flight;
- a high ball reachable only near its ends;
- control at the target;
- separate reaction times for the passing team and the defenders;
- optionally, the target projected along the receiver's run.

It is fitted by likelihood on four IDSSE matches.

| Completion | v1 availability (recalibrated) | Probabilistic model, projected target | SkillCorner's own model |
|---|---|---|---|
| IDSSE, 3 held-out matches: log-loss / AUC | 0.297 / 0.719 | 0.296 / **0.762** | — |
| SkillCorner, 20 matches, fitted on the league: log-loss / AUC | 0.424 / 0.781 | **0.370 / 0.842** | 0.279 / 0.896 |
| SkillCorner held out, refitted on its own matches | — | **0.321 / 0.859** *(leaks the outcome; without height 0.356 / 0.838)* | 0.283 / 0.894 |

- **The structure was the gap, not the constants.** It closes about half the AUC gap to SkillCorner's model when transferred from the league, and two-thirds when refitted *(about half without SkillCorner's height flag, which leaks the outcome)*. Projecting the receiver along their run is the largest single gain.
- **It is a probability:** refitted on a provider, predicted and observed completion agree within about 0.06 in every band. Fitted on the league it over-predicts SkillCorner's completion by about 6 points, so a published probability needs fitting per provider, or on pooled data.
- **Not established:** that height helps (the two datasets disagree), that the measured player speeds help, or the parameters as physical quantities. *(Corrected 19 November 2026: refitted on SkillCorner without the height flag the model scores 0.356 / 0.838 held out, against 0.321 / 0.859 with it, so height helps on SkillCorner; it is on the league's data that it does not.)* *(Withdrawn the same day: SkillCorner logs `high_pass` on completed passes only, so the gain is the outcome leaking in. Height is not established; see "Pass height through Matrix" below.)* One consistent pattern: defenders effectively start 0.5 to 0.75 s after the passing team.
- **Not adopted; v1 stays.**
- **Proposed for the founder:** a "pass completion probability" metric (0.1.0) beside v1, labelled estimated, fitted per provider. It would serve as:
  - the availability half of the option split;
  - the risk term of a narrower decision quality version 2 (v1's end-of-pass credit plus the risk at the pass).

## Pass completion probability adopted (19 November 2026)

**Founder decision:** the probabilistic model ships as the **pass completion probability** (`pass_probability` 1.0.0): a new metric beside v1, labelled estimated, with parameters fitted per provider.
- **Parameter sets:** SkillCorner's own (fitted on its 10 even matches), and a default fitted on the league's tracking for every other provider. The Data check marks the metric Degraded where the default is used.
- **The shipped path, checked end to end on SkillCorner's 10 held-out matches** (7,749 passes): log-loss 0.327, **AUC 0.843**, predicted against observed completion within about 0.04 in every band.
- **In the semantic events (schema 1.2):** `pass_completion_probability`, `pass_completion_status` and `pass_completion_parameters` ([ai-layer.md](ai-layer.md#the-semantic-football-event)).
- **No probability without a logged target.** Most providers log none for a failed pass (Metrica logs none; SkillCorner logs one for most incomplete passes), and Plaiq's inferred receiver is not used.
- **Follow-ups:**
  - ~~carry a provider's pass-height flag through Matrix and refit with it (worth 0.035 log-loss on SkillCorner)~~ done; the refit is not adopted because the flag leaks the outcome (below);
  - fit sets for Metrica and STATSports once data with failed-pass targets exists;
  - use the metric as the option split's availability half and the risk term of a narrower decision quality version 2 *(the risk term measured 19 November 2026, below)*.
- **Found on the way:** Matrix's Metrica CSV loader leaves second-half event coordinates unrotated (Games 1 and 2). The metric avoids it by using tracked positions; the loader fix is a separate decision ([status.md](status.md)). *(Fixed 19 November 2026.)*

## Pass height through Matrix (19 November 2026)

**Built:** Matrix's events carry an optional `High Pass` flag (`True`, `False` or unknown). It comes from SkillCorner's `high_pass` and StatsBomb's `pass_height` ("High Pass" only). Metrica logs no height, and none is inferred.

**Found: SkillCorner's flag records the outcome.** It is present on completed passes only: 986 high and 11,821 not high, all of them completed, and no failed pass carries it.
- **The earlier "height helps on SkillCorner" (0.356 to 0.321) was this leak.** A missing flag was read as "not high", so every high pass in the fit was completed.
- **The shipped path showed it again.** A with-height set took held-out log-loss from 0.327 to 0.305, almost all of it on high passes, which are all completed.
- **Where height is logged on every pass** (the league's data), it made no difference.

**Decision taken:**
- **No with-height parameter set.** The pass completion probability does not read the flag (`pass_probability` stays 1.0.0, and a test pins it).
- **The column stays** as a description of a pass, documented as known only for completed passes on SkillCorner, and never an input to a model of the outcome.
- **Height stays not established.** It could still come from the tracking itself for providers that track ball height.

Evidence: `ponta-platform/PLAIQ_REFERENCES_REPORT.md`, "Pass height through Matrix".

## Decision quality version 2 with the pass completion probability as its risk term (19 November 2026)

Two candidates in `analytics/decision_quality_v2.py` 0.2.0, beside version 1 and 0.1.0, which are unchanged. Both are measured on the same 12,112 SkillCorner passes and groundings.
- **v1 with risk:** version 1's score times the chosen pass's completion probability.
- **Projected:**
  - each team-mate's option is a pass into their run, worth its completion probability times its value there;
  - the score is the chosen option over the best one;
  - one formula covers completed and failed passes, and the outcome is never an input.

| | Version 1 | v1 with risk | **Projected** | 0.1.0 |
|---|---|---|---|---|
| Optimal share: completed / failed | 58% / 89% | 31% / 23% | **31% / 27%** | 30% / 19% |
| Shot, completed passes, AUC | 0.60 | 0.42 | **0.62** | 0.51 |
| Shot, all passes, AUC | 0.55 | 0.44 | **0.62** | 0.53 |
| Rank correlation with SkillCorner's analogue | **0.31** | 0.09 | 0.29 | 0.15 |
| Predicts completion, AUC | 0.23 | 0.64 | **0.51** | 0.69 |

- **v1 with risk fails:** it becomes a measure of safety and reverses the link to chances. Dropped.
- **Projected is the first version 2 to keep version 1's signal and remove its flaw.**
  - It is level with or better than version 1 on chances, held on both halves.
  - It is close on SkillCorner's analogue.
  - Failed passes are no longer lenient, and it neither rewards nor penalises safety.
- **Not established:**
  - the goal link (89 goals; it falls on the held-out half);
  - the untuned bands on its 0 to 1 range.
- **It inherits** Plaiq's threat surface for value, and pass completion's per-provider fit and need for a logged target.
- **Not adopted; version 1 stays the default.** Projected (0.2.0) is a candidate for the founder: decision quality version 2 beside version 1, labelled estimated. If adopted, failed-pass decision quality could be reported for the first time. *(Corrected 19 November 2026: version 1 already reports a score for most failed passes, the lenient one above. What projected adds is a failed-pass score that is not lenient. Adopted the same day, below.)*

Evidence: `ponta-platform/PLAIQ_REFERENCES_REPORT.md`, "Decision quality version 2 with the pass completion probability as its risk term".

## Decision quality version 2 adopted beside version 1 (19 November 2026)

**Founder decision:** projected ships as **decision quality version 2** (`decision_quality_v2` 1.0.0, the 0.2.0 maths unchanged), beside version 1, labelled estimated. **Version 1 stays the default** decision quality (`dq_score`, `dq_band`).
- **In the engine:** `MatchAnalysis.decision_quality_v2`, one row per pass the pass completion probability covers. It reads the same moment of the pass (the tracked ball, else the passer; the players at the start frame), so the chosen pass's probability is the shipped pass completion probability.
- **In the semantic events (schema 1.3):** `dq_v2_score`, `dq_v2_status`, `dq_v2_best_option_ref` and `dq_v2_version` ([ai-layer.md](ai-layer.md#the-semantic-football-event)). A failed pass that version 2 scores now gets a row even where version 1 has no failed-pass score.
- **Scale: 0 to 1, no bands.** 1.0 means the best pass on offer was chosen. Version 1's bands are cut on its 0 to 1.5 scale and are not applied to version 2; bands for version 2 wait until they are measured and agreed.
- **Not scored, with a reason:** goalkeepers and set pieces (as version 1), and any pass without a completion probability (no logged target, which is most failed passes on Metrica, or no tracking at the pass).
- **In the Data check:** a 14th metric, "Decision quality v2 (expected value)". Degraded where pass completion uses the default parameter set, and, like version 1, where the events are derived from tracking.
- **Checked end to end:** the measurement tool now reads the shipped property and reproduces the measured scores on SkillCorner's 20 matches (`ponta-platform/PLAIQ_REFERENCES_REPORT.md`, "Decision quality version 2 adopted").
- **Still not established:** the goal link and any bands. Cite a version 2 score as an estimate, beside version 1, and never alone as a verdict on a player.

## The models, what tests them now, and what to change

| Model | How it works today | What can test it now | Proposed change |
|---|---|---|---|
| **1v1 pressure** (1 to 10) | Hex-ring distance to the nearest opponent only | The league's pressure on each shot (IDSSE); SkillCorner's opponents within 5 m at reception; a blind judged sample against FIFA's three levels | Add the presser's angle (in front counts more than behind) and the number of pressers, as FIFA's EFI pressure does. Band the result into direct, indirect and no pressure, calibrated on a judged sample; this also settles the open "named high-pressure threshold" |
| **Threat surface** (xT) | Geometric: goal-mouth angle × distance decay + a sigmoid progression bonus, normalised | Empirical goal probability by zone from Wyscout's 3.25 million events; SkillCorner's threat per pass; the league's expected goals per shot | A data-driven threat surface learned from Wyscout's possessions (the published expected-threat method), compared with the geometric one against SkillCorner's threat and shot outcomes |
| **Control and interception** (targeted PCZ) | Kinematic arrival times with default player physics (9 m/s, 5.5 m/s², 0.7 s reaction) and a linear ball-speed model (5 m/s plus 0.2 m/s per metre) | Real pass completions (calibration and ranking); SkillCorner's expected pass completion; measured ball speeds and reception times in the league's 25 Hz tracking | Measure the ball-speed model and reaction time against real passes before changing anything. If they are off, calibrate them from data, scoped to the adult professional population as engine governance requires. *(Measured 19 November 2026: the ball speed was off and is now measured; the constants are not where the completion gap lies. See "Step 6 results".)* |
| **Pass option value** | xT × control × (1 − interception risk) × progressive factor × (1 − distance decay); hand-set factors | SkillCorner's passing-option lists, targeted player and "dangerous" flags; the founder's judged decisions | Split *availability* (control and interception) from *value*. Add unit awareness: the number of opposition lines the pass would break (P5), as FIFA values penetration. Replace the hand-set progressive factor with the data-driven threat surface |
| **Decision quality** | POV of the actual receiver at the end of the pass over the best POV at the start; bands 0.55 / 0.80 / 0.95 | A blind judged sample (the replay method used for phases and possession); SkillCorner's targeted player against its options | Measure first: the expert agreement study engine governance already plans. Then the denominator should be the best *available* option, not the best of every team-mate |
| **Failed-pass inference** | Walks the ball's line past the interception to the first team-mate who would have controlled it | SkillCorner's targeted player on unsuccessful passes | Score candidate targets by the pass's direction, speed, height and the runs being made, and validate against SkillCorner's targeted player. Keep the current inference for its present use until the new one is measured |
| **Carry decision quality, movement value** | Ring alternatives and ΔPOV | The judged sample | Measure with the decision-quality study |

## Proposed programme, in order

Each step is measured against the references with fit and held-out matches, and follows [engine-governance.md](engine-governance.md):
- a new formula is a **new versioned metric beside the base**;
- **the base stays the default** until the expert study supports the change;
- nothing is called accurate until then.

1. **A Plaiq reference harness** (measurement only). Run Plaiq on SkillCorner's 20 open-data matches and the seven IDSSE matches, and report:
   - failed-pass target accuracy against SkillCorner's targeted player;
   - control and interception risk against real completions and SkillCorner's expected completion;
   - pass option value against SkillCorner's threat and "dangerous" flags;
   - pressure against the league's shot pressure;
   - the threat surface against Wyscout's empirical goal probability by zone.

   This gives every later step its baseline. It needs no engine change.
2. **Failed-pass target inference, version 2.** It unblocks line breaks for failed passes and makes failed-pass decision quality more defensible.
3. **Option availability.** A team-mate is available when control and interception risk pass thresholds fitted to SkillCorner's option lists. It unblocks line-breaking options and gives decision quality a cleaner denominator.
4. **Pressure version 2** (angle and count, three FIFA-style bands), calibrated on a judged sample.
5. **A data-driven threat surface** from Wyscout, compared with the geometric one.
6. **Kinematics calibrated from the league's 25 Hz tracking** (ball speed, reaction time), after step 1 shows whether they need it. *(Done 19 November 2026: ball flight and player speeds measured; reaction time not calibratable in the current formula; next is a probabilistic interception and control model. See "Step 6 results" above.)*
7. **Unit-aware option value:** line-breaking (P5) as a component or as context for decision quality ("was a line-breaking option available, and was it taken?").
8. **The decision-quality expert study**, run blind with the replay method on matches the bands were not tuned on. The founder's judgements can pilot it; analysts complete it for Gate 3b.

## Limits

- **SkillCorner's and the league's labels are their own models,** not truth. Agreement with them shows consistency with an industry reference, not accuracy. FIFA publishes definitions, not data, so FIFA alignment is by definition only.
- **The specification lives in the frozen prototype.** Engine changes need a production copy of `ANALYTICS_SPECIFICATION.md` in `ponta-platform`, with the changelog discipline engine governance describes, before the first formula changes.
- **The physical defaults are tuned for adult professional players** ([engine-governance.md](engine-governance.md#player-population-validity-scoping)). Calibration from the league's data keeps that scope.
- **Public wording stays as it is:** "a defined, explainable score", with no claim that decision quality is accurate or predictive, until the study reports.

## Candidate bands for decision quality version 2 (19 November 2026)

Version 2 shipped 0 to 1 with no bands. Four cut-off sets were measured on the same 12,095 scored SkillCorner passes. The fitted sets were chosen on the even matches only; the table is the held-out odd matches. **Measured, not agreed.**

| Candidate | Cut-offs | Shot rate by band, poor to optimal | Rises in held-out match resamples |
|---|---|---|---|
| Version 1's cut-offs | 0.55 / 0.80 / 0.95 | 3.2%, 6.3%, 7.6%, 9.3% | 56% |
| Meaning (half, three-quarters, the best) | 0.50 / 0.75 / 0.95 | 3.1%, 5.4%, 7.8%, 9.3% | 80% |
| Fitted | 0.45 / 0.60 / 0.90 | 2.8%, 4.9%, 6.4%, 9.4% | 94% |
| **Fitted, top anchored** | **0.45 / 0.60 / 0.95** | **2.8%, 4.9%, 6.9%, 9.3%** | **94%** |
| *Version 1's bands on version 1* | 0.55 / 0.80 / 0.95 | 1.8%, 1.4%, 5.3%, 8.8% | 35% |

- **Every candidate rises band by band on both halves;** they differ in how reliably. Version 1's own bands do not rise on the held-out half.
- **The overall ranking is the same for all** (band AUC 0.60). The cut-offs move the steps, not the signal.
- **The steps:** the shot rate climbs to about 5% by 0.5 to 0.6, then flattens from about 0.7. Above 0.6 the data separates little except the best option itself.
- **No failed-pass leniency in any candidate** (top anchored: optimal 29% of failed passes against 31% of completed; version 1: 88% against 59%).
- **Goals do not separate** (31 goals on the held-out half).
- **Recommended for the founder: the top-anchored set (0.45, 0.60, 0.95).** It is as stable as the free fit, keeps optimal as "the best option, within 5%", and puts the lower steps where the shot rate climbs. The meaning set is simpler to explain but less reliable between suboptimal and good.
- **To decide with it:** whether version 2's bands reuse version 1's names. With different cut-offs on a different scale, the same names invite merging the two.
- **Limits:** one provider and one outcome (a shot in the possession, which credits a pass for what follows it). Bands describe the score; they do not validate it.

Evidence: `ponta-platform/PLAIQ_REFERENCES_REPORT.md`, "Candidate bands for decision quality version 2".
