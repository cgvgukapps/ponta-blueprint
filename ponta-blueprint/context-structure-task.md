# Context Structure (P2, P3, P4) — Execution Task

> **Status: built (17 November 2026); Gate 2a bars and thresholds agreed by the founder the same day; zones passed the spot check (47 of 50); phases revised after the founder's replay review (P4 0.2.0) and roles revised (P2 0.2.0), both 18 November 2026, with every new threshold agreed; restarts from positions added the same day (P4 0.3.0, its four throw-in thresholds agreed).** Stage B of [context-packages.md](context-packages.md), started under decision 4a (proceed without waiting for analysts). Built in `ponta-platform` (`plaiq_engine/context/structure.py`, `zones.py`, `phase.py`); results in `CONTEXT_STRUCTURE_REPORT.md` there.
>
> - **Reference match (1886347):** roles agree with SkillCorner's by position group for 79% of playing time; phases at Cohen's κ 0.60.
> - **All 20 SkillCorner open-data matches** (measured afterwards, nothing tuned):
>   - formations meet the bar everywhere;
>   - phases hold pooled (κ 0.61) but clear the per-half bar on only 11 matches;
>   - roles reach a median 75.4% and clear the bar on 12 matches, so they do not generalise.
> - **After the revisions (18 November 2026):**
>   - roles reach a median 92.2% and meet the grouped bar on 19 matches and the merged bar on 17 (most earlier misses were the comparison misreading SkillCorner's LM and RM, its central pair);
>   - phases pool at κ 0.61 and meet the founder's set bar pooled, with two matches just under the 0.50 floor (after P4 0.3.0);
>   - all four bars are met on 10 of 20 matches.
>
> Gate 2a is met on the reference match, zones included. Across the set it is met for formations and for phases pooled, and not yet on five matches (three for roles, two for the phase floor).
>
> **Gate 2a round (20 November 2026):** both remaining gaps diagnosed (`CONTEXT_STRUCTURE_REPORT.md`, "The Gate 2a round").
> - **The phase floor** fails because of the own-third build-up rule. A quarter of the pitch clears it everywhere (lowest half 0.524) and lifts pooled κ to 0.670 / 0.638.
> - **Roles** miss on 1959846 because its home team plays a box midfield P2 has no template for; a template for it costs more than it fixes.
> - **Agreed by the founder the same day:** the quarter boundary (P4 0.4.0, thresholds 0.12.0) and a set-level reading for roles (pooled 84.1% grouped, 89.0% merged). **Gate 2a is met across the set**: phases pooled κ 0.670 / 0.638 with no match below 0.50, roles pooled above both bars, formations everywhere. Stage C: [context-relationships-task.md](context-relationships-task.md).

Self-contained task for a coding agent working in the `ponta-platform` monorepo. It builds context packages P2 (team structure), P3 (relative zones) and P4 (phase of play) on the framework from [context-foundations-task.md](context-foundations-task.md) (complete: context grid, registry, Data check entries, P1 possession and game state). Read [context-packages.md](context-packages.md) first.

## Context

Stage A gives every 0.2 s grid point a possession label (home, away, contested, dead ball, unknown) with its source, game state and possession chains. Stage B adds what a coach means by "how a team played": its shape and lines, where the ball and attackers are relative to the opponent's lines, and the phase of play. Under decision 4a the stage proceeds on **reference agreement** (Gate 2a); analyst agreement (Gate 2b) still gates player-facing use.

**Coordinates.** `NormalisedMatch` positions are metres, centred, home attacking +x for the whole match. For team-relative work, rotate each team so it attacks +x with its own goal line at x = 0. For the away team, mirror both axes, so left is always +y. Checked on SkillCorner's player roles: the home left-back's mean y is +17.5 m, the right-back's −16.4 m; for the away team the signs flip. Pitch size comes from `match.pitch` (Metrica 106 × 68 m, SkillCorner 104 × 68 m); thirds are a third of the length.

**References for Gate 2a** (comparison only, decision 2; read without player names):

- SkillCorner 1886347: a role for every player (GK, LB, LCB, CB, RCB, RB, DM, LDM, RDM, AM, LW, RW, CF) with substitution times, and SkillCorner's phase of play on every player-possession event (in possession: build_up, create, finish, direct, quick_break, transition, set_play, chaotic, disruption; out of possession: high_block, medium_block, low_block and their defending variants). About 25% of grid time carries a phase label.
- Metrica Games 1 to 3: logged set pieces (corner kicks, free kicks, throw-ins, kick-offs).
- Published definitions and values, below.

## Research basis (summary; full brief with sources kept in the session record)

- **Shape measures** (outfield players): length (deepest to highest), width, centroid, stretch index (mean distance to centroid), convex-hull area, defensive line height (deepest outfield player to own goal). Reference values in official matches: length 31–46 m, width 35–48 m, hull about 900 m², line height 22–28 m (Rico-González et al. 2021). Defence–midfield and midfield–attack line distances are 10.2 ± 3.8 m and 13.3 ± 4.2 m (Forcher, Bundesliga).
- **Lines:** 1D k-means (k = 3) on players' mean x per window, separately in and out of possession (Forcher and Goes). A line's height is its deepest member (FIFA EFI); inter-line distance is between line centroids.
- **Formation and roles:** match players' window-mean positions, centred and scaled, to formation templates with the Hungarian algorithm (Bialkowski et al. 2016; Wei et al. 2013). Use 2-minute windows per possession state and drop possessions under 5 s (Shaw and Glickman 2019). Declare a formation change only if it persists, for example 5 min (SoccerCPD, Kim et al. 2022). Expert agreement in the literature is 72–75% for formation and 86.5% for positions.
- **Relative zones:** "between the lines" and "behind the line" are relative to the opponent's lines (FIFA EFI, using the deepest player of a line). Five lanes by convention on the penalty-area and goal-area edges (no provider publishes boundaries). "The pocket" has **no published definition**; Ponta defines it as between the lines, in a half-space and inside the opponent's hull.
- **Phases:** SkillCorner's published rules (16 February 2025 spec):
  - build_up: ball in the own third with pressure or a high opponent;
  - create: the default;
  - finish: final or middle third, defensive line near the box, at least 1 s;
  - direct: a long ball of at least 32 m in x from the own half;
  - quick_break / transition: a regain in the opponent's or own half followed by rapid progression;
  - set_play: corners, free kicks and long throws into the box, up to 20 s;
  - chaotic: a possession under 5 s with fewer than 3 passes.

  Blocks come from the defending team's deepest three outfield players (own third: low; middle third: medium). FIFA EFI's mid-block test is less than 10 km/h, at least 8 outfield players in the middle third, held 1.6 s of 2.0 s. Pressure, "rapid progression" and "close to the box" are not published; they are Ponta's thresholds and are measured.
- **Human agreement on phases** is about κ 0.7 (Krippendorff α 0.68 across analysts, Kuroda et al. 2026; Fleiss κ 0.73, Hands et al. 2023). A rule-based labeller should not be expected to beat that against any single reference.

## Objective

Three packages on the existing registry, each with the full contract (definitions, outputs, Data check entry, tests, golden outputs, versions), and measured agreement with the references so the founder can set the Gate 2a bars.

## Hard constraints

Those of [context-foundations-task.md](context-foundations-task.md) carry over:
- additive only;
- the import rule holds;
- no coordinates for the AI;
- no names or licensed data in the repo;
- deterministic;
- every field defined;
- thresholds in the versioned config with a status;
- measure before proposing.

In addition:
- Provider labels never become outputs.
- Every label without a published basis is named as a Ponta choice in its definition.

## Deliverables

### P2 Team structure (depends on P1)

- **Per frame and team:** length, width, centroid depth, stretch index, convex-hull area, defensive line height, the three line heights (deepest member) and the two inter-line distances, and outfield players observed. The goalkeeper (the deepest player on average) is excluded.
- **Per window** (2 min, per possession state, possessions of at least 5 s, cut at period ends): line membership by 1D k-means, the formation template that fits best with its margin over the second best ("unclear" below the agreed margin), and a role per player from the template slot. Written as a segment table, and as each team's current formation on every frame.
- **Data check:** Degraded when outfield players observed fall below the agreed share. Coverage is the share of frames with a shape.

### P3 Relative zones (depends on P1, P2)

- **Per frame**, relative to the team out of possession:
  - the ball's lane (five lanes) and third in the attacking team's frame;
  - whether the ball is in front of, between or behind the opponent's lines;
  - whether it is inside the opponent's hull, and whether it is in the pocket;
  - counts of attacking players between the lines, behind the line and in the pocket.
- **Data check:** inherits P2's state; the ball-relative fields are Degraded where the ball is undetected.
- **A spot-check page** of 50 sampled frames (pitch drawing plus labels) for the founder's review, the Gate 2a check for zones.

### P4 Phase of play (depends on P1, P2)

- **A deterministic state machine** on the grid: dead ball, set play, quick break or transition, direct, chaotic, then established play (build_up, create, finish), with SkillCorner's pairing for the team out of possession (high, medium and low block and the defending variants). Also an independent line-based block label from the deepest three defenders.
- **Hysteresis:** phases shorter than the agreed minimum merge into their neighbour.
- **Data check:** coverage is the share of in-play time labelled; Degraded where possession was inferred.

### Measurement and report

- **P2 roles:** each player's most common role against SkillCorner's role, exact and by position group (full-back left and right, centre-back, defensive mid, central mid, attacking mid, winger left and right, forward).
- **P2 formations:** the most common formation per team and half on every match, with stability (changes per half).
- **P4 phases:** a confusion matrix and Cohen's κ against SkillCorner's in-possession phases on grid points where both give the same team in possession. Agreement of set_play with Metrica's logged corners and free kicks.
- **Sensitivity:** sweeps over the phase thresholds that have no published value.
- **Runtime and storage** per match.

## Acceptance criteria

- All three packages run on all four regression matches through the registry, write version-stamped columns and segment tables, and attach Data check entries.
- Golden outputs are captured and a second run matches.
- Measurements exist for every item above; proposed bars and thresholds cite them.
- The full local gate passes, existing outputs are unchanged, and the import check and lint pass.
- No player name, licensed data or coordinate array reaches the repo or any AI-facing field.

## Explicitly not in scope

Stage C packages; the AI layer; changing P1 or the engine formulas; analyst studies (Gate 2b); ingesting provider labels as outputs.

## Final report (`CONTEXT_STRUCTURE_REPORT.md`, repo root)

What was built; the measurements; proposed thresholds and Gate 2a bars with evidence; where and why labels disagree with the references; runtime and storage; deviations; decisions for the founder.
