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

## The models, what tests them now, and what to change

| Model | How it works today | What can test it now | Proposed change |
|---|---|---|---|
| **1v1 pressure** (1 to 10) | Hex-ring distance to the nearest opponent only | The league's pressure on each shot (IDSSE); SkillCorner's opponents within 5 m at reception; a blind judged sample against FIFA's three levels | Add the presser's angle (in front counts more than behind) and the number of pressers, as FIFA's EFI pressure does. Band the result into direct, indirect and no pressure, calibrated on a judged sample; this also settles the open "named high-pressure threshold" |
| **Threat surface** (xT) | Geometric: goal-mouth angle × distance decay + a sigmoid progression bonus, normalised | Empirical goal probability by zone from Wyscout's 3.25 million events; SkillCorner's threat per pass; the league's expected goals per shot | A data-driven threat surface learned from Wyscout's possessions (the published expected-threat method), compared with the geometric one against SkillCorner's threat and shot outcomes |
| **Control and interception** (targeted PCZ) | Kinematic arrival times with default player physics (9 m/s, 5.5 m/s², 0.7 s reaction) and a linear ball-speed model (5 m/s plus 0.2 m/s per metre) | Real pass completions (calibration and ranking); SkillCorner's expected pass completion; measured ball speeds and reception times in the league's 25 Hz tracking | Measure the ball-speed model and reaction time against real passes before changing anything. If they are off, calibrate them from data, scoped to the adult professional population as engine governance requires |
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
6. **Kinematics calibrated from the league's 25 Hz tracking** (ball speed, reaction time), after step 1 shows whether they need it.
7. **Unit-aware option value:** line-breaking (P5) as a component or as context for decision quality ("was a line-breaking option available, and was it taken?").
8. **The decision-quality expert study**, run blind with the replay method on matches the bands were not tuned on. The founder's judgements can pilot it; analysts complete it for Gate 3b.

## Limits

- **SkillCorner's and the league's labels are their own models,** not truth. Agreement with them shows consistency with an industry reference, not accuracy. FIFA publishes definitions, not data, so FIFA alignment is by definition only.
- **The specification lives in the frozen prototype.** Engine changes need a production copy of `ANALYTICS_SPECIFICATION.md` in `ponta-platform`, with the changelog discipline engine governance describes, before the first formula changes.
- **The physical defaults are tuned for adult professional players** ([engine-governance.md](engine-governance.md#player-population-validity-scoping)). Calibration from the league's data keeps that scope.
- **Public wording stays as it is:** "a defined, explainable score", with no claim that decision quality is accurate or predictive, until the study reports.
