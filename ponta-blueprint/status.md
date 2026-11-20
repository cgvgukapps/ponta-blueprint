# Status and handoff

**Last updated:** 19 November 2026. This is the living pick-up note: read it first when resuming, and update it at the end of any session that changes state. It describes the present; the history of how it got here is in git and in each repo's notes. For the plan see [roadmap.md](roadmap.md); for an outside reader's summary see [platform-overview.md](platform-overview.md).

**One-line state:** pre-product, no customers. The engines, the Data check and the semantic builder are built and tested on real open data in `ponta-platform`; the AI layer, API, worker, storage and production web app are not built. A clickable UI prototype on mock data has been through two redesigns and brief v2 items 5.1 to 5.4. A single-file match page built from real engine output is ready for the founder-run concierge step. **Strategy, 17 November 2026:** Ponta is repositioned as a match context model that the AI explains; context packages (possession to episodes) come before the AI build ([context-packages.md](context-packages.md)). Context Stages A and B are built (Gate 1 passed; Gate 2a met on the reference match, zones 47 of 50 on the founder's spot check). Phases were revised after the founder's replay review (P4 0.2.0) and roles after the 20-match check (P2 0.2.0); Restarts are found from positions without an event feed (P4 0.3.0; its four throw-in thresholds agreed). Across SkillCorner's 20 open-data matches, Gate 2a is met pooled for phases and on most matches for roles, with five matches short of a bar. Next: discovery calls (gate E1), the concierge step and a Stage C spec, with the rest of Phase 1 alongside.

## Repositories

| Repo | Local path | Remote (GitHub, `cgvgukapps`) | Position |
|---|---|---|---|
| Blueprint (this repo) | `/Users/cicerovieira/PontaPlan` | `ponta-blueprint` | `main`; tag `blueprint-baseline-2026-11-15` |
| Prototype (frozen) | `/Users/cicerovieira/PONTA-PROTOTYPE` | `ponta-prototype` | `main` at `41bd812`, tag `prototype-validation-final`. Never modify. |
| Platform (production) | `/Users/cicerovieira/ponta-platform` (on disk: `Ponta-Platform`) | `ponta-platform` | `main` at `f4ac82a` (Gate 2a closed: build-up within a quarter of the pitch, P4 0.4.0, thresholds 0.12.0; roles judged on the set; 20 November 2026), pushed; before it `de0d6dd` and `15503c4` (the Gate 2a round), `ceca992` (Gain version 2 adopted beside version 1, schema 1.6); `c1919d2` (Gain and Choice: version 1's bands renamed, schema 1.5; one-model Gain measured); `557e002` (decision quality v2 1.1.0 adopted: hex cells, ring 1, with agreed bands; schema 1.4); `e39fb4c` (hex-cell candidates measured); `6c9a2c1` (decision quality v2 candidate bands measured, not agreed); `48e35db` (decision quality v2 adopted beside v1, projected, 1.0.0, schema 1.3); `52f255d` (decision quality v2 0.2.0, pass completion as the risk term, measured); `da92464` (pass-height flag carried through Matrix; the with-height refit not adopted, the flag leaks the outcome); `dad9fe9` (Metrica CSV second-half event coordinates fixed); `94c4316` (pass completion probability adopted); `8a90507` (probabilistic pass model measured); `8a31d60` (Plaiq calibration, step 6); `b74808a` (decision quality v2 measured, not adopted); `c26319b` (decision quality grounded); `359a7fd` (learned threat surface measured, not adopted); `a0565ea` (pressure v2 measured, not adopted); `baa9967` (Plaiq option split); `6f084d2` (Plaiq reference harness); `a589424` (context P5 0.1.0, line breaks); `fcc1efe` (line-break prototype and its 20-match results); `d33c8ab` (loose-ball rule downstream checks); `3143f45` (the loose-ball rule, built and off, thresholds 0.9.0); `26db3cf` (reference data: IDSSE and Wyscout); `8251e93` (throw-in thresholds agreed, 0.8.0); `61e2bc8` (restarts from player positions, P4 0.3.0) and `64301e0` (founder's re-check of the seven changed replays); before it `8895f89` (role revision, P2 0.2.0, thresholds 0.6.0); before it `d14a56c` (founder's phase review, corner rule, P1 and P4 0.2.0) and `0263e9f` (Stage B thresholds agreed, 20 open-data matches; Game 2 fix at `f284563`) |
| UI prototype | `/Users/cicerovieira/Ponta_Platform_UI` | `ponta-platform-ui` | `main` at `8e9dfec` (Gain and Choice: version 1's band names, 19 November 2026), pushed; before it `26b7c1d`; tag `prototype-v0.1` at `4bd8e52` |
| Marketing website | `/Users/cicerovieira/Ponta-Website` | `ponta-website` | `main` at `51f2956` (illustrative card pressure on the engine's 1 to 10 scale, D6; 20 November 2026), pushed (pushing `main` deploys the site, still `noindex, nofollow`); before it `067c482` (Gain band names, D5 and C30), `82a3429` |

## Where things stand

### Engines (`ponta-platform`)

- **Extraction complete** ([platform-extraction-task.md](platform-extraction-task.md), `EXTRACTION_REPORT.md`): `matrix_engine` and `plaiq_engine` as installable packages, all prototype tests carried over, import-graph check, CI.
- **Tests:** 1,143 in total; 1,131 in the full local suite and 12 in the opt-in equivalence test (`PONTA_EQUIVALENCE=1`, about 7 minutes). CI is green but skips data-dependent tests; the local gate is `PONTA_REQUIRE_SAMPLE_DATA=1 pytest -q`.
- **Equivalence:** Plaiq fed through Matrix reproduces the prototype's golden outputs on Metrica Sample Game 3 and SkillCorner 1886347, except 2 of 39,589 decision rows and 12 of 36,792 alternative rows on Metrica (a characterised float near-tie). No band changes. Accepted as the baseline; `engine_version` unchanged.
- **Data check** (`matrix_engine/parity.py`): 15 metrics in 6 families with reasons, measured figures and coverage (pass completion probability and decision quality v2 added 19 November 2026, Gain version 2 on 20 November 2026; the decision metrics are labelled Gain and Choice). On the real SkillCorner match, pass option value and event ΔV are Full and shot quality is Blocked.
- **Semantic builder** (`plaiq_engine/semantic_builder.py`): one pseudonymised row per action, no coordinates; [ai-layer.md](ai-layer.md#the-semantic-football-event) matches the implemented schema.
- **Tools:** `tools/demo.py` (end to end on a sample match) and `tools/match_page_payload.py` (payload for the shareable match page). Golden outputs are regenerable (`tools/golden/README.md`).
- **Open:** the unused `prev_event` parameter of `resolve_identities`. Facade decisions are parked (`docs/MATCH_ANALYSIS_INVENTORY.md`). `apps/api`, `apps/worker` and `apps/web` are empty.

### UI prototype (`ponta-platform-ui`, mock data only)

- Vite, React and TypeScript with a fake in-browser engine, synthetic tracking and hand-written sample brief and answer text. Not connected to any backend.
- **Brief v2:** 5.1 (client-side names), 5.2 (coverage), 5.3 (shareable match page) and 5.4 (review fixes) are done; 5.5 (API mapping) is on hold by founder decision. Ask is also a panel inside each match.
- **Design:** decisions are fixed in [frontend-design-brief-v2.md](frontend-design-brief-v2.md#3-fixed-decisions-do-not-reopen-without-reporting-why) section 3, with the reasoning in the UI repo's `docs/design-notes.md` (sections 6 to 17). Current look: cool near-black palette, serif page headings, white primary buttons, Ponta blue only for focus, selection marks and graphics, grey highlights for selected rows.
- **Shareable match page:** a single offline HTML file from real engine output (SkillCorner 1886347: 1,308 of 1,371 passes and carries scored, 95.4%). Real payloads are generated outside the repos.
- **Open:** Safari print not verified (no page numbers there; `<thead>` repetition unchecked). The UI still shows pressure as a 0 to 1 index with 0.7 as "high"; align it with the engine's 1 to 10 scale once the threshold is chosen.

### Website (`ponta-website`)

- v2 one-product page deployed with `noindex`; hero "Tactical & physical AI that sees the whole pitch, and the athlete inside it."; Sign in disabled.
- Launch waits on gates G1 to G6 ([website-v2-plan.md](website-v2-plan.md#launch-gates-public-announcement-not-the-friends-only-url)): licences (G3), legal pages and sub-processor terms (G4) and the hero's fusion wording (G6) are the substantive ones. Also open: `/security`, `/privacy`, `/terms`, and hero artwork that still shows two engines.

### Blueprint (this repo)

- Reviewed and cleaned on 17 November 2026: status banners on every task spec and plan, stale facts corrected (pressure scale, pitch rendering, product colours, provider list, website state), the old company name "PLAIQ" replaced by Ponta where it meant the business, British spelling throughout, links checked.
- **Context Stage B built (17 November 2026)** in `ponta-platform` ([context-structure-task.md](context-structure-task.md); report `CONTEXT_STRUCTURE_REPORT.md`): P2 team structure (shape, lines, formations, roles), P3 relative zones (lane, third, band against the defending lines, hull, pocket) and P4 phase of play (SkillCorner's vocabulary, Ponta's rules). Roles agree with SkillCorner's by position group for 79% of playing time; phases at Cohen's κ 0.60 (0.58 and 0.61 by half). About 16 s and 4 MB per match for P1 to P4. The founder agreed the Gate 2a bars and every Stage B threshold the same day (thresholds 0.4.0).
- **Stage B on SkillCorner's 20 open-data matches (17 November 2026):** MIT licence, kept outside the repos at `~/ponta-reference-data/skillcorner-opendata`; measured with `tools/context_reference_set.py` and nothing tuned.
  - **Formations:** meet the bar on all 20 matches.
  - **Phases:** pooled κ 0.61 (0.62 and 0.59 by half), but only 11 of 20 matches clear κ 0.55 on both halves; none falls below 0.50.
  - **Roles:** median 75.4% by playing time; only 12 of 20 matches clear 75%. Most misses are wide midfielders given central slots (44%) or centre forwards given an attacking-midfield slot (15%).
  - **Summary:** Gate 2a is met on the reference match, and not across the set for roles.
  - **Closed 20 November 2026:** after the role revision (P2 0.2.0), the restarts round (P4 0.3.0) and the Gate 2a round (build-up within a quarter of the pitch, P4 0.4.0; roles judged on the set), **Gate 2a is met across the set**: phases pooled κ 0.670 / 0.638, no match below 0.50; roles pooled 84.1% grouped, 89.0% merged; formations 20 of 20.
- **P5 0.1.0, line breaks, built (18 November 2026)** in `plaiq_engine/context/line_breaks.py`.
  - **Covers:** completed passes; failed passes listed with their breaks unknown.
  - **Threshold:** `p5_min_unit_players` (2, FIFA's definition), proposed.
  - **Its own output against SkillCorner:** any break κ 0.65 / 0.63, furthest line κ 0.53 / 0.54, through or around 87%.
- **Plaiq reference harness (18 November 2026)**, step 1 of the review (`PLAIQ_REFERENCES_REPORT.md` in `ponta-platform`):
  - **completion:** AUC 0.78, against SkillCorner's own model at 0.90;
  - **pass option value:** rank correlation 0.28 with SkillCorner's threat; AUC 0.53 for its "dangerous" flag;
  - **pressure** against the league's shot pressure: 0.30 (1v1), 0.49 (zonal);
  - **threat surface:** 0.54 against the league's xG; 0.81 against Wyscout's shot goal rate by zone.
  - **Failed-pass targets cannot be tested on SkillCorner** (its end point is its own target), which also corrects the earlier line-break explanation.
- **Plaiq option split (18 November 2026):** pass option value written as availability × value (`analytics/option_split.py` 0.1.0, beside POV).
  - **Availability** predicts completion at AUC 0.77 held out (POV 0.66).
  - **Value** tracks SkillCorner's threat at 0.78 (POV 0.26).
  - **POV's weak reference scores came from multiplying the two.**
  - **Passing options are offers to receive** (movement), so line-breaking options move to P6.
- **Pressure version 2 (18 November 2026):** directional and count-aware, after FIFA and Andrienko et al.
  - **Measured and not adopted:** Plaiq's 1v1 pressure matches SkillCorner's pressure levels best (κ 0.80), and zonal matches the league's shot pressure best.
  - **Gave a measured candidate for the named high-pressure threshold** (8 or more on the 1v1 scale).
- **Learned threat surface (18 November 2026):** expected threat learned from 970 Wyscout matches.
  - **On held-out Wyscout matches** it wins on scoring within 10 s.
  - **On SkillCorner's threat, shot goal rates and the league's xG and goals,** it loses slightly to Plaiq's geometric surface.
  - **Not adopted; the geometric surface stays.** POV's value gap lies in its progressive factor and distance decay.
- **Decision quality grounded (18 November 2026), not judged**, at the founder's direction. Measured against SkillCorner's option values, outcomes and context on 12,012 passes.
  - **Completed passes have outcome evidence:** shots 1.2% at poor rising to 9.1% at optimal, on par with SkillCorner's analogue.
  - **Plaiq rates 89% of failed passes optimal** (a flaw).
  - **Weak agreement with SkillCorner on the best option** (0.31).
  - **Next proposed:** decision quality version 2 as expected value from the option split.
- **Decision quality version 2 (19 November 2026):** expected value at the pass.
  - **Measured and not adopted:** it fixes the failed-pass leniency but becomes a risk measure, and loses the link to chances (0.51 against 0.60).
  - **Version 1 stays.** Ball-speed calibration comes first; failed-pass decision quality stays unreported.
- **Plaiq calibration, step 6 (19 November 2026):** targeted pitch control's physics measured on the seven IDSSE matches (`analytics/ball_flight.py` 0.1.0, equal to v1 by default; `tools/plaiq_calibration.py`; fit on four matches, tested on three).
  - **Ball speed:** real passes are 30% to 40% faster than v1 assumes up to 30 m. Measured: 8.5 m/s plus 0.165 m/s per metre, which halves the timing error held out (median 0.42 s to 0.20 s). The ball slows along a pass.
  - **Players:** median top speed 8.4 m/s and peak acceleration 4.7 m/s², against v1's 9.0 and 5.5.
  - **Completion ranking barely moves:** 0.719 to 0.739 held out, 0.781 to 0.793 on SkillCorner (its own model 0.896), and only with a reaction time of 0 s, which is not physical. The gap is in the availability formula's structure (no ball height, reach or control time; not a probability), not its constants.
  - **Not adopted; v1 stays.** Proposed next: a probabilistic, height-aware interception and control model using the measured physics.
  - **IDSSE reader fixed:** a freshly parsed match reached Plaiq with no players (only cached loads had them).
- **Probabilistic pass model (19 November 2026):** `analytics/pass_probability.py` 0.1.0, after Spearman (2018), beside targeted pitch control ([plaiq-review.md](plaiq-review.md)). Fitted on four IDSSE matches.
  - **SkillCorner, transferred:** log-loss 0.424 to 0.370, AUC 0.781 to 0.842.
  - **SkillCorner, refitted on its own matches:** 0.356 / 0.838 without height, against SkillCorner's own model at 0.283 / 0.894. *(The 0.321 / 0.859 reported earlier used SkillCorner's height flag, which leaks the outcome; see finding 10.)*
  - **IDSSE held out:** AUC 0.719 to 0.762, log-loss level.
  - **Projecting the receiver along their run is the largest gain.** Height and measured player speeds are not established.
  - **It is a calibrated probability when fitted per provider.**
  - **Adopted by the founder (19 November 2026)** as the pass completion probability (`pass_probability` 1.0.0), beside v1, labelled estimated, fitted per provider (SkillCorner's own set; a default from the league's tracking for others, Degraded in the Data check). Shipped in `MatchAnalysis.pass_completion` and semantic events schema 1.2. End to end on SkillCorner's held-out matches: log-loss 0.327, AUC 0.843, calibrated within about 0.04 by band. No probability without a logged target (all of Metrica's failed passes).
- **Decision quality v2 with pass completion as its risk term (19 November 2026):** `decision_quality_v2.py` 0.2.0, two candidates beside version 1, measured on 12,112 SkillCorner passes ([plaiq-review.md](plaiq-review.md)).
  - **v1 × completion probability fails:** it becomes a measure of safety (shot AUC 0.42).
  - **Projected** (each team-mate's pass into their run, completion probability × value, chosen over best) keeps version 1's signal: shot AUC 0.62 against 0.60 on completed passes, 0.62 against 0.55 on all; SkillCorner analogue 0.29 against 0.31.
  - **It removes the failed-pass leniency:** 27% optimal against 89%.
  - **It is neutral on completion** (AUC 0.51).
  - ~~**Not adopted; version 1 stays.** A candidate for the founder (open decisions).~~
  - **Adopted by the founder (19 November 2026)** as decision quality version 2 (`decision_quality_v2` 1.0.0, the 0.2.0 maths), beside version 1, labelled estimated. Version 1 stays the default.
    - **Shipped as:** `MatchAnalysis.decision_quality_v2`; semantic events schema 1.3 (`dq_v2_score`, `dq_v2_status`, `dq_v2_best_option_ref`, `dq_v2_version`); a 14th Data check metric.
    - **Scale:** 0 to 1, no bands; never on version 1's bands. *(Bands agreed later the same day; below.)*
    - **Not scored:** goalkeepers, set pieces, and passes without a completion probability. A failed pass it scores now gets a semantic row even where version 1 has none.
    - **Checked end to end:** the measurement tool reads the shipped property and reproduces the measured scores on SkillCorner's 20 matches.
  - **Revised the same day (founder decision): 1.1.0 on hex cells, with bands.** Each team-mate's option is the best of the 7 cells around where their run reaches when the ball arrives (ring 1); it replaces the point-based 1.0.0. Bands: Best option 0.95 and above, Near best 0.60 to 0.94, Below best 0.45 to 0.59, Well below best under 0.45 (`dq_v2_band`, schema 1.4), descriptive and distinct from version 1's, in neutral colour steps. Version 1 stays the default ([plaiq-review.md](plaiq-review.md)).
- **Plaiq review (18 November 2026):** [plaiq-review.md](plaiq-review.md).
  - **Evidence:** Plaiq's maths is tested, but its football validity is not.
  - **Two building blocks fell short this week:** failed-pass intent did not help line breaks (its cause is not yet established, since SkillCorner's data cannot test it); option value is not an availability test.
  - **The plan:** an ordered, versioned programme starting with a reference harness that measures every model against SkillCorner, the league's shot pressure and xG, and Wyscout.
- **Line breaks, a prototype for Stage C (18 November 2026):** a rule on FIFA's definition, measured against SkillCorner's labels on 20 matches (6,628 passes).
  - **Lines:** grouped at the pass moment, which beat P2's window lines on fit and held-out matches.
  - **Agreement:** any line broken κ 0.68 / 0.66; furthest line κ 0.52; through or around 82%.
  - **Plaiq's failed-pass intent does not help** (κ 0.27 to 0.26). The cause is not established: on SkillCorner's data a failed pass's end point is already SkillCorner's own target.
  - **Line-breaking options:** the geometry agrees (κ 0.63), but Plaiq's option value is not an availability test.
  - **Founder decisions:**
    - bring completed-pass line breaks into P5;
    - a target rule for failed passes (in the context layer, or a Plaiq change);
    - an availability model for options.
- **FIFA alignment (18 November 2026):** [fifa-alignment.md](fifa-alignment.md) compares Ponta with FIFA's Football Language (117 term pages) and EFI metrics.
  - **Same approach:** units and phases by rules on tracking.
  - **Already matching:** line height, team length, compactness, five channels, between-units receiving location.
  - **Gaps:** in-contest, line breaks, FIFA's phase splits (opposed and unopposed build-up, press against block, counter-press), and the manually coded concepts.
  - **No public FIFA claim** without legal advice.
- **New reference data (18 November 2026)**, downloaded outside the repos with `tools/fetch_reference_data.py` (checksums and a `SOURCE.json`; licences in [legal-compliance.md](legal-compliance.md#open-reference-datasets-added-18-november-2026)):
  - **IDSSE** (`~/ponta-reference-data/idsse`, 2.6 GB). Seven 2022/23 German league matches with official TRACAB tracking, events, playing positions and line-ups, and frame-level official ball status and possession. CC BY 4.0: credit the DFL and cite Bassek et al. (2025). Read by `tools/idsse.py` as a reference reader, not a supported provider.
  - **Wyscout open events** (`~/ponta-reference-data/wyscout-opendata`, 80 MB). 1,941 matches, 3.25 million events, 2017/18. CC BY 4.0. Events only; the players file is never opened.
- **Findings from them** (`CONTEXT_STRUCTURE_REPORT.md`):
  - **P1 against the official frame-level possession: 82% to 89%** (median 88%), below Gate 1's 93%, which was measured against event-inferred possession. Disagreement concentrates just after P1 switches possession and on short passes.
  - **The league logs restarts 1 to 3 s before the ball is played.**
  - **Main roles agree with official positions at a median 91%**, and reading roles from the defensive shape is confirmed here too.
  - **Restarts from positions on full-pitch tracking:** corners 93% found and 67% right; throw-ins 83% and 79%.
  - **Possession from events alone:** a median 85% against the official reference, so tracking adds about 3 points.
  - **P1's possession starts match Wyscout's norms** over 1,941 matches. P1 finds fewer, longer possessions than an event rule (247 against about 300 to 350 a match).
  - **P4's breaks** agree with the league's counter-attack flag when Ponta labels one (5 of 7), but catch only about a third of the flagged counter-attacks (5 of 14 shots).
- **Zone spot check (17 November 2026):** the founder judged 47 of 50 zone cards right (bar 45), so zones meet Gate 2a. Phases on the same still cards scored 34 of 50, with finish weakest at 9 of 17. A still frame rarely shows a phase, so the phase check was redone as 7-second replays (`tools/context_spotcheck.py --replay`).
- **Founder's phase replay review and refinements (17 and 18 November 2026):** 29 of 50 replays judged right, with notes; kept in `ponta-platform/tools/data/phase_review_2026-11-17.json`.
  - **Three P4 refinements adopted** (P4 0.2.0): finish needs the ball in the attacking half, build-up whenever the ball is in the own third, a break ends in the final third. Each was scored on the review and against SkillCorner's 20 matches, and kept only if it lost nothing there. Three more were tried and dropped.
  - **Corners from player positions** (the founder's idea: one attacker at a flag, a crowd in the box), used without an event feed: 73% of SkillCorner's held-out corners and 69% of Metrica's, against 4% and 23% for the ball rule; about 55% of detections are right.
  - **P1 bug fixed** (P1 0.2.0): a set piece was credited to whichever team touched the ball first, so a cleared corner made the defenders' chain a set play.
  - **Result:** review 34 of 50 (28 before, with the corrected P1); the founder re-judged the seven changed replays on 18 November 2026 and found all seven right, so 35 of 50 (`tools/data/phase_review_2026-11-18_recheck.json`); SkillCorner κ pooled 0.610 (0.609 and 0.611 by half); per match, 13 of 20 clear 0.55 on both halves (11 before), but three now sit just under 0.50.
  - The six new thresholds were agreed by the founder on 18 November 2026 (thresholds 0.6.0).
- **P2 role revision (18 November 2026, P2 0.2.0):**
  - **Most role misses were in the comparison, not Ponta.** SkillCorner files LM and RM under "Midfield" (the central pair of its 4-4-2) and LF and RF under "Wide Attacker"; the comparison had read them as wide midfielders and centre forwards. Corrected from SkillCorner's own position groups, confirmed by mean positions.
  - **Each player's main role is now read from the team's defensive shape** (out-of-possession windows, else all windows), published as P2's `main_roles` segment. Teams SkillCorner lists as 4-4-2 often fit a 4-2-3-1 in possession. Adopted after it improved on half the matches and held on the other half.
  - **Result:** roles median 92.2% (75.4% before); 19 of 20 matches meet the grouped bar and 17 the merged bar. On the reference match the grouped measure is 77.0% (79.3% before) and the merged 95.2% (90.5%). One match fell, 72% to 59%, where the defensive shape fits a back three.
- **Phase bar for a set of matches (founder, 18 November 2026):** pooled κ at least 0.55 on each half (met, 0.609 and 0.611) and no match below 0.50. Three matches fell short of the floor by up to 0.02 and count as failing; after P4 0.3.0, two (2013725 at 0.482, 2017461 at 0.490).
- **Restarts round (18 November 2026, P4 0.3.0):** without an event feed, throw-ins are found from positions (one player beyond the touchline, play slowed, the ball near): 55% found and 61% right on held-out SkillCorner matches, 60% and 87% on Metrica. Goal kicks cannot be: 73% of SkillCorner's have a tracking gap before them. SkillCorner does not call throw-ins or goal kicks set plays, so they are published as restart facts (a P4 `restarts` segment), a throw-in no longer starts a break, a throw-in set-up gets no phase and a corner set-up is `set_play`. Review 36 of 50 (the founder confirmed the throw-in moment as right with no phase); SkillCorner κ 0.610 (phases file 0.619 to 0.625). The four throw-in thresholds were agreed by the founder the same day (thresholds 0.8.0).
- **Gate 2a across the 20 matches:** formations 20, roles 19 (grouped) and 17 (merged), phase κ 0.55 on each half 13; all four bars on 10. Met on the reference match. Short of a bar: five matches (roles on 1959846, 2006229, 2016236; the phase floor on 2013725, 2017461).
- **Context Stage A built (17 November 2026)** in `ponta-platform` (`plaiq_engine/context/`): the context grid, package registry, Data check entries from packages, golden harness, and P1 possession and game state on all four regression matches; semantic events schema 1.1 adds possession chain, source, score and red-card difference. Report: `CONTEXT_FOUNDATIONS_REPORT.md`.
- **Context foundations spec written (17 November 2026):** [context-foundations-task.md](context-foundations-task.md). It uses a regression set of four local matches and treats SkillCorner's own possession, phase and game-state fields (found in its dynamic events) as comparison data only.
- [platform-overview.md](platform-overview.md) is a dated snapshot for an independent reviewer; also published as a private page for the founder.
- **Context packages adopted (17 November 2026):** [context-packages.md](context-packages.md) records the founder's work plan, adjusted after review, and four decisions: (1) repositioning agreed, the AI build waits for Stage A at the earliest, preferably Stage B; (2) Ponta computes its own context, provider metrics are stored for comparison only; (3) unvalidated fields may reach analysts labelled "estimated, not yet validated", never player-facing views; (4) only Stage A (P0, P1) is committed, Stages B to D planned, P9 in Phase 5. Data-tiers, AI layer, roadmap, risks (27 to 30), go-to-market, competitive landscape and engine governance updated to match.

## Findings that change the plan

1. **`roster/` belongs in `plaiq_engine`, not `matrix_engine`.** Matrix never imports it, and it imports Plaiq's `DEFAULT_PROFILE`. `pitch_config` belongs in `matrix_engine` (both engines import it). Recorded in `ponta-platform/docs/ARCHITECTURE.md`.
2. **`compute_event_pov` row order is non-deterministic across runs; values are deterministic.** The semantic builder sorts explicitly.
3. **The ball is undetected in 32.2% of Metrica Game 3 frames and 26.4% of the SkillCorner match.** The engine returns an empty control grid there; coverage reports it.
4. **Decision quality ranges from 0 to 1.5**, not 0 to 1. Bands: Poor below 0.55, Suboptimal 0.55 to 0.79, Good 0.80 to 0.94, Optimal 0.95 and above. *(19 November 2026: this is version 1, the default. Version 2, adopted beside it and labelled estimated, runs 0 to 1 with its own descriptive bands: Best option 0.95 and above, Near best 0.60 to 0.94, Below best 0.45 to 0.59, Well below best under 0.45.)*
5. **SkillCorner event ids are inconsistent** (`From` is `'1'`, `To` is `'1.0'`). Normalised in the new code.
6. **Metrica Sample Game 2 was normalised with the home team attacking −x** (the home goalkeeper's mean x was +39.7 m): the Metrica CSV loader flipped the second half and assumed the first-half direction. **Fixed 17 November 2026** (platform `f284563`): both Metrica CSV loaders now rotate a match whose home goalkeeper defends +x. Only Game 2's recorded attack signs changed in the context golden outputs; no prototype parity reference covers Game 2. Engine outputs on Game 2 that depend on direction (decision quality, attacking-third labels) now differ from the prototype's.
7. **Single-source matches have no per-frame confidence** (only `ball_detected` and `players_observed`), which leaves the confidence slot of an AI citation undefined for Tier 1 (see open decisions).
8. **UI prototype:** its fake engine and mock packs must never reach a production bundle, and its hand-written types must become generated types from the API schema.
9. **Metrica CSV second halves: event coordinates were not rotated with the tracking** (found and fixed 19 November 2026, platform `dad9fe9`). The shared Matrix helper, inherited from the prototype, sliced the events table by a tracking frame label, so second-half events of Metrica CSV games (Sample Games 1 and 2, and any CSV export through the Metrica adapter) sat about 55 m from the ball. Tracking is bit-identical after the fix.
   - **What changed, second halves of Games 1 and 2 only:** P5 line breaks (undercounted before: 44 to 155 and 29 to 134), Plaiq's ΔV, pass option value and decision quality, and `pitch_zone`.
   - **What did not:** P1 to P4, Game 3, SkillCorner, and every recorded measurement (the bands were audited on Game 3).
   - **The frozen prototype keeps the defect.**
10. **SkillCorner's `high_pass` flag is logged on completed passes only** (found 19 November 2026 while carrying it through Matrix). The count is 986 high and 11,821 not high, all completed; no failed pass carries it. So the flag records the outcome.
   - **Withdrawn:** "height helps on SkillCorner" (0.356 to 0.321). A with-height set on the shipped path showed the same leak (0.327 to 0.305, almost all on high passes, which are all completed).
   - **Not shipped:** the with-height parameter set. Pass completion does not read the flag, and a test pins it.
   - **Kept:** Matrix's events carry `High Pass` (SkillCorner, StatsBomb; Metrica logs none), documented as never an input to a model of the outcome.
   - **The league's data,** where height is logged on every pass, showed no gain from it.

## Next steps, in order

0. ~~**Website: carry the Gain band names.**~~ Done 20 November 2026 in `Ponta-Website` `067c482` (launch-draft D5, ledger C30), pushed: band names and the Gain label on the worked examples, the meters show each segment's starting score. The illustrative card's "opposition pressure of 0.84" (a 0 to 1 index) now reads "8.4 out of 10", on the engine's 1 to 10 scale (`Ponta-Website` `51f2956`, launch-draft D6, ledger C12; 20 November 2026, pushed).
1. **Track V (validation), now.** Discovery calls with the wedge, intercepted-pass and price questions (gate E1, at least 8 calls); check each prospect's vendor licence before taking a real export ([legal-compliance.md](legal-compliance.md#vendor-licence-check-before-the-first-real-export)).
2. **Gate 2a closed (20 November 2026); build Stage C next.**
   - **The Gate 2a round (20 November 2026, `CONTEXT_STRUCTURE_REPORT.md`):**
     - **Phases:** the two matches below the 0.50 floor (2013725, 2017461) come from the own-third build-up rule. Narrowing it to a quarter of the pitch (`p4_build_up_max_depth_share` 0.25) lifts pooled κ to 0.670 / 0.638 (from 0.609 / 0.610), holds on the held-out matches, leaves no match below 0.50 (lowest 0.524) and 18 of 20 at κ 0.55 on both halves, and loses one of the founder's 50 judged replays (Metrica 1 grid point 25371). **Agreed by the founder 20 November 2026** (P4 0.4.0, thresholds 0.12.0).
     - **Roles:** 1959846's home team defends in a 4-2-2-2 (box midfield), which P2 has no template for. A 4-2-2-2 template was measured and not adopted (it costs the set more than it fixes). Pooled over 583 players roles agree 84.1% grouped and 89.0% merged. **Agreed the same day:** roles are judged on the set, as phases are (pooled at least 75% grouped and 85% merged, met), with 1959846 recorded as a known limit.
     - **Gate 2a is met across the set** on the agreed settings: phases pooled κ 0.670 / 0.638, no match below 0.50 (lowest 0.524), 18 of 20 at κ 0.55 on both halves; roles pooled 84.1% / 89.0%; formations 20 of 20.
   - **Review items moved to Stage C:** long balls and losses in flight, and a goal kick credited while the ball is undetected, need in contest from logged events (P1 0.3.0). Recycling backwards and a short regain stay as notes for Gate 2b.
   - **Stage C, deliverable 1 built (20 November 2026):** P1 0.3.0 adds FIFA's possession states (`p1_state`) beside the unchanged possession, with in contest from logged duels and clearances, and chains gain "interrupted" and out-of-play outcomes. Against the founder's 50 possession calls it is a wash (21 right, as before): the event log cannot place most "nobody has it yet" moments, which need a separate ball-state field (recommended). `p1_contest_max_s` (4 s) for the founder to agree.
   - **Stage C spec written:** [context-relationships-task.md](context-relationships-task.md) (P5 relationships, P6 movement types on FIFA's categories, P7 space and context-adjusted Choice, in contest from events, a FIFA view of phases), each measured against SkillCorner's line-break, run-type, goal-side and pressing labels.
   - **Plaiq:** done since this list was written. The reference harness ran; option availability is the pass completion probability; Choice scores failed passes from their logged target. Stage C reuses them.
   - Analyst agreement (Gate 2b) is still needed before anything is player-facing.
3. **Concierge step** (roadmap Phase 2, step 0): run a prospect's real export locally through the engines and send back the Data check and the match page (gate E2). The page and payload tool exist.
4. **Rest of Phase 1** (not gated by E1), alongside context Stage A: Supabase (Postgres, RLS, Auth), FastAPI skeleton, object storage, the automated cross-tenant access test for Parquet paths, basic ToS ([roadmap.md](roadmap.md#phase-1--platform-foundation-and-engine-extraction)).
5. **Before the AI layer** ([ai-workflows-task.md](ai-workflows-task.md)): context Stages A and B are built; decide the citation confidence for single-source matches and a named high-pressure threshold; choose the LLM provider and put sub-processor terms in place before any real customer data reaches a model.
6. **Frontend, later:** import the UI into `ponta-platform/apps/web` (subtree, or copy and record the source commit); keep the `src/api` signatures and add an HTTP client behind a mock switch; generate types from the OpenAPI schema. The Data check is the first screen that can go real. The API mapping (brief v2, 5.5) waits until then.

**Running in parallel:** the legal gates in [roadmap.md](roadmap.md#track-l--legal-and-compliance-runs-alongside-each-item-gated-to-a-trigger) (basic ToS before the first pilot; LLM sub-processor terms before real customer data reaches a model); the website launch gates above; optionally a `soccertrack-validation-task.md` and the Tier 3 gate status in [data-tiers.md](data-tiers.md) (offered, not requested).

## Open decisions

- **Citation confidence for single-source matches:** what fills the confidence slot (a documented derivation from `ball_detected` and `players_observed`, or an explicit null with the reason). Blocks the AI layer.
- **A named high-pressure threshold** on the engine's 1 to 10 scale, documented and versioned; the UI glossary then follows. *Measured candidate (18 November 2026):* 1v1 pressure of 8 or more is "high", from cut points 3 / 6 / 8 / 10 that reproduce SkillCorner's five pressure levels (weighted κ 0.80 held out). For FIFA's coded levels: direct ≈ 8 or more, indirect ≈ 3 to 7, none ≈ 1 to 2. To confirm on a judged sample ([plaiq-review.md](plaiq-review.md)).
- **Context P1 refinements (open, with recommendations):** a control radius per provider (once a second broadcast match confirms SkillCorner's fit at 2.0 to 2.5 m), a hand-labelled possession sample before Stage B, and movement-based gap inference when a feed needs it ([context-packages.md](context-packages.md#open-decisions)).
- **Context package thresholds:** grid rate, inferred-possession share, possession agreement bar (Gate 1), and the later ones per package; listed in [context-packages.md](context-packages.md#open-decisions), each decided before its package starts, from measurements where possible.
- **Recruit one or two analysts** through discovery calls for context Gates 2 and 3 (formation, phase, movement labels).
- **Evidence-gate thresholds (E1 to E3)** in [roadmap.md](roadmap.md#evidence-gates-proposed-15-november-2026-founder-to-confirm-the-thresholds): confirm or change.
- **Lead wedge** (single-export brief, or fragmentation and provider transition): decided by discovery calls ([go-to-market.md](go-to-market.md#two-wedges-one-to-lead-with-added-15-november-2026)).
- **Provider list on the website:** the page lists Second Spectrum as supported; the engine has it fixture-tested only and the hard rule allows four providers. Change the page or revise the rule with a reason.
- **LLM provider and model** (blocks Phase 2's AI work).
- **Whether event-only analysis ships in the first release.**
- **Ask Ponta pricing:** capped, metered or bundled.
- **Hosting:** Fly.io, Render or Railway for the services, the frontend possibly static on Cloudflare. Recommendation given: managed services, plain containers, EU regions from the start, no AWS now.
- **Vite or Next.js** for the production frontend (leaning Vite static export).
- **Facade decisions** (`MatchAnalysis` input form, rendering helpers, private-name wrappers): parked; recommendations in `ponta-platform/docs/MATCH_ANALYSIS_INVENTORY.md`.
- **Public naming** of Matrix and Plaiq; the IP-protection call on publishing `ANALYTICS_SPECIFICATION.md`.

*Decided and no longer open:* Gate 2a closed across the set: build-up within a quarter of the pitch, and roles judged on the set (20 November 2026); Gain version 2 (the one-model Gain, 2.0.0) adopted beside version 1, labelled estimated, with version 1's cut-offs and names measured for it; the honeycomb as the platform's spatial language, with no cell indices in any LLM context (20 November 2026); the display names Gain (version 1) and Choice (version 2), and version 1's descriptive band names Full gain, Most of the gain, Partial gain, Low gain at the same cut-offs, applied to the platform, the blueprint and the UI prototype (19 November 2026); decision quality version 2 on hex cells (ring 1, 1.1.0) with its bands (0.45 / 0.60 / 0.95, named Best option, Near best, Below best, Well below best), separate scales for the two versions and no merging into one number (19 November 2026); decision quality version 2 (projected), adopted beside version 1, labelled estimated, 0 to 1 with no bands (19 November 2026); the pass-height flag through Matrix (carried; the refit not adopted because SkillCorner's flag leaks the outcome; 19 November 2026); the Metrica CSV loader fix (19 November 2026); the pass completion probability metric, adopted beside v1, labelled estimated and fitted per provider (19 November 2026); the four P4 0.3.0 throw-in thresholds and the review's throw-in moment (18 November 2026); the six P4 0.2.0 thresholds, the P2 role revision and the phase bar for a set of matches (18 November 2026); the Gate 2a bars, every Stage B threshold, the Metrica Game 2 direction fix and the SkillCorner open-data download (17 November 2026); proceeding through the context stages against reference labels, with analyst gates kept for player-facing use (decision 4a, 17 November 2026); the context-packages repositioning and its four decisions, the Gate 1 bar (met) and all P1 thresholds (17 November 2026); the extraction baseline and the SkillCorner Data check (15 November 2026); the UI palette, away colour, heat gradient, primary button style and canvas pitch rendering (fixed in [frontend-design-brief-v2.md](frontend-design-brief-v2.md#3-fixed-decisions-do-not-reopen-without-reporting-why) section 3).

## Docs that now disagree with the evidence

None known after the 17 November 2026 review. Older sections of `architecture.md` and `website.md` still say "Next.js"; each carries a note that the framework is undecided, rather than being rewritten.

## Strategy gaps found but not acted on

The wearable-integration question (OAuth connections, webhooks, PlayerData, Garmin, Apple Watch, Catapult, STATSports Sonra, session-based sync) is **not** in the strategy beyond file upload, Vault-stored credentials and polling. No action requested. If revisited: add OAuth and token-paste connector types, an optional webhook receiver, and a session entity next to matches, all gated behind the Phase 4 health-data legal review. Apple HealthKit is on-device, so it would need a companion app. Vendor access details in the proposal were not verified.

## How to resume

```bash
# Engines
cd /Users/cicerovieira/ponta-platform
source .venv/bin/activate            # recreate if missing, see README
python tools/check_imports.py && ruff check packages tools tests && pytest -q
# Context measurements (write outside the repo; the cache holds pickled sample matches)
python tools/context_golden.py --out /tmp/ponta-context/golden --cache /tmp/ponta-context/cache --check
python tools/context_structure_measure.py --cache /tmp/ponta-context/cache --out /tmp/ponta-context/structure.json
python tools/context_reference_set.py --data ~/ponta-reference-data/skillcorner-opendata --cache /tmp/ponta-context/sc-cache --out /tmp/ponta-context/reference_set.json
python tools/context_phase_review.py --cache /tmp/ponta-context/cache --sc-data ~/ponta-reference-data/skillcorner-opendata --sc-cache /tmp/ponta-context/sc-cache --out /tmp/ponta-context/phase_review.json
python tools/context_corner_sweep.py --cache /tmp/ponta-context/cache --sc-data ~/ponta-reference-data/skillcorner-opendata --sc-cache /tmp/ponta-context/sc-cache --out /tmp/ponta-context/corner_sweep.json
python tools/fetch_reference_data.py idsse            # and: wyscout --dest ~/ponta-reference-data/wyscout-opendata
python tools/context_idsse.py --data ~/ponta-reference-data/idsse --cache /tmp/ponta-context/idsse-cache --out /tmp/ponta-context/idsse.json
python tools/context_event_chains.py --cache /tmp/ponta-context/cache --idsse ~/ponta-reference-data/idsse --idsse-cache /tmp/ponta-context/idsse-cache --wyscout ~/ponta-reference-data/wyscout-opendata --out /tmp/ponta-context/event_chains.json
python tools/context_restart_sweep.py --cache /tmp/ponta-context/cache --sc-data ~/ponta-reference-data/skillcorner-opendata --sc-cache /tmp/ponta-context/sc-cache --out /tmp/ponta-context/restart_sweep.json

# UI prototype
cd /Users/cicerovieira/Ponta_Platform_UI
npm run dev                          # http://127.0.0.1:5173
npm run typecheck && node scripts/check-i18n.mjs && npm run contrast && npm run build
npm run shots                        # with the dev server running; add "-- --codes" for codes mode
```

Prototype tests use the prototype's own interpreter: `cd /Users/cicerovieira/PONTA-PROTOTYPE && PYTHONDONTWRITEBYTECODE=1 .venv/bin/python -m pytest -q -p no:cacheprovider` (about 2 minutes, expect 964 passed). Regenerate golden outputs: see `tools/golden/README.md` in `ponta-platform`.

## Gotchas

- **CI skips the sample-data tests,** so a green CI does not cover the whole suite. Run `PONTA_REQUIRE_SAMPLE_DATA=1 pytest -q` locally before relying on a change. `gh` is not installed; read runs with `curl https://api.github.com/repos/cgvgukapps/ponta-platform/actions/runs`.
- **Install both engine packages in one pip command,** so the local packages resolve each other and pip never looks up an internal name on the public index (distribution names are prefixed `ponta-`).
- **Do not commit `*.egg-info`** (git-ignored).
- **Context golden summaries are git-ignored** (derived from sample data); only `tests/golden/context/manifest.json` is tracked. After changing a package or threshold version, recapture with `tools/context_golden.py --capture`, or the golden test in `tests/cross_engine/test_context_sample.py` skips.
- **Do not rename or move blueprint documents:** other repos link to them by name (see CLAUDE.md).
- **Folder case:** the platform folder is `Ponta-Platform` on disk; macOS ignores case, so `ponta-platform` also works. The venv records the capitalised path.
- **Piping a long job through `grep` hides all output until it finishes,** and the shell reports the pipe's exit status, not the job's.
- **UI screenshots need the dev server running** and use the system Chrome; real match-page payloads are generated outside the repos and never committed.
- **Do not touch the prototype** beyond reading it. It is the validation record.
- **SkillCorner's open data (20 matches, about 2 GB) lives at `~/ponta-reference-data/skillcorner-opendata`,** outside every repo. Tracking files are Git LFS objects, fetched from `media.githubusercontent.com`. The regression match 1886347 is still read from the prototype's older copy; the newer copy's revised event file gives κ 0.59 instead of 0.60.
- **IDSSE parsing takes about 30 s per match** (370 MB of XML each); `tools/idsse.py` caches parsed matches as pickles and rebuilds the P1 event table on every load. Its restart times are logged 1 to 3 s early; score restarts against the official ball status turning live.
- **Measurement tool caches hold pickled matches.** Delete a match's pickle after changing a loader, or the tools keep using the old normalisation.
