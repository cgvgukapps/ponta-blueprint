# Context Relationships, Movement and Space (P5, P6, P7) — Execution Task

> **Status: in progress (20 November 2026).** Deliverable 1 built (P1 0.3.0), a ball-state field added at the founder's request (P1 0.4.0), deliverable 2 built (P5 0.2.0), deliverable 3 built and revised after the founder's run review (P6 0.2.0), the ball state refined after the founder's check (P1 0.5.0), deliverable 4 built (P7 0.1.0, thresholds 0.19.0 proposed); see `CONTEXT_RELATIONSHIPS_REPORT.md`. Next: the founder's decisions on P7, then deliverable 5 (the FIFA view of phases). Stage C of [context-packages.md](context-packages.md). Written after the Gate 2a round; the founder closed Gate 2a across the set the same day ([status.md](status.md)). It does not wait for Gate 2b (analysts), under decision 4a.

Self-contained task for a coding agent working in the `ponta-platform` monorepo. It completes context package P5 (relationships), builds P6 (movement types) and P7 (space, value and game-state context), and adds two smaller pieces that FIFA's definitions call for: in-contest possession from logged events (P1 0.3.0) and a FIFA-vocabulary view of phases (P4). Read [context-packages.md](context-packages.md) and [fifa-alignment.md](fifa-alignment.md) first.

## Context

**What exists.**
- **Stage A** (P0 grid at 0.2 s, P1 possession, chains and game state) and **Stage B** (P2 shape, lines, formations and roles; P3 zones relative to the opponent's lines; P4 phases on SkillCorner's vocabulary) are built in `plaiq_engine/context/`, with the registry, Data check entries, versioned thresholds (`context/thresholds.py`) and golden outputs. Reports: `CONTEXT_FOUNDATIONS_REPORT.md` and `CONTEXT_STRUCTURE_REPORT.md` in the repo root.
- **P5 0.1.0, line breaks** (`context/line_breaks.py`): completed passes only. The defending team's units are grouped at the pass moment, each with at least two players (FIFA). A pass breaks a unit when it goes from in front of the unit's deepest player to beyond it, through or around. Against SkillCorner's labels on 20 matches: any break κ 0.65 / 0.63 by half, furthest line κ 0.53 / 0.54, through or around 87%.
- **Plaiq, adopted since Stage B** (`plaiq_engine/analytics/`, `MatchAnalysis`):
  - the **pass completion probability** (`pass_probability` 1.0.0): a pitch-control model after Spearman (2018), parameters fitted per provider; the chance, known at the moment of the pass, that a pass to a point is completed;
  - **Choice** (decision quality version 2, 1.1.0): every team-mate's option is the best of the 7 hex cells around where their run reaches when the ball arrives; the score is the chosen option's expected value over the best one's, 0 to 1;
  - **Gain version 2** (2.0.0): the value realised where the ball arrived over Choice's best expected value.
  These are reused as they are. **Do not re-fit or change them**; report discrepancies.
- **The honeycomb is the platform's spatial language** (decided 20 November 2026, [architecture.md](architecture.md#storage-hybrid-by-access-pattern)). Plaiq's hexagonal grid has one square metre per cell. Spatial outputs are stored, aggregated and drawn by cell and rolled up to P3's zones. Models compute continuously where the physics needs it, then snap to cells. They evaluate only cells selected for a reason, never every cell (about 7,460 per pitch).

**Coordinates.** As in [context-structure-task.md](context-structure-task.md): metres, centred, home attacking +x; rotate each team to attack +x with its own goal line at x = 0 and left at +y.

**References** (comparison only, founder decision 2; read without player names; kept outside every repository):
- **SkillCorner open data**, 20 matches of the 2024/25 A-League (MIT licence), `~/ponta-reference-data/skillcorner-opendata`. Its dynamic events carry, per player possession and per passing option:
  - line-break flags (`first_line_break`, `second_last_line_break`, `last_line_break`, `furthest_line_break` with type) and `n_opponents_bypassed`;
  - counts of line-breaking passing options (`n_passing_options_line_break` and by line);
  - `goal_side_start` and `goal_side_end`, `dangerous`, `overall_pressure_start` and `_end`;
  - **off-ball runs** typed in ten subtypes (`behind`, `coming_short`, `dropping_off`, `support`, `overlap`, `underlap`, `pulling_wide`, `pulling_half_space`, `run_ahead_of_the_ball`, `cross_receiver`), with `break_defensive_line` and `intended_run_behind`;
  - `on_ball_engagement` events and **pressing chains** (`pressing_chain`, its length and end type).
- **IDSSE**, the German league's official tracking and events (7 matches), `~/ponta-reference-data/idsse`: logged duels, clearances, blocks, interceptions and the official frame-level possession.
- **Metrica** Sample Games 1 to 3 (logged events) and **Wyscout** open events.
- **The founder's judged moments:** the phase replay review (`tools/data/phase_review_*.json`) and the possession replay check on IDSSE (`CONTEXT_STRUCTURE_REPORT.md`, possession round).

**Two review cases this stage should fix.** In the phase review a long ball intercepted in flight and a ball travelling after a loss were labelled for the team that had lost it. The cause is P1 holding the old team while the ball travels. The founder kept the tracking-based loose-ball rule off (`p1_loose_on_opponent_touch`); FIFA takes "in contest" from logged events, which is deliverable 1 below. *(Corrected 20 November 2026, after deliverable 1: the two long-ball cases are a P4 phase question; the founder accepts `direct` there, and a pass in flight is in possession under FIFA's definitions too. In contest from events also misses most of the founder's "nobody has it yet" possession calls, which need a separate ball-state field: see the report.)*

## Research basis (summary)

- **FIFA's Football Language** ([fifa-alignment.md](fifa-alignment.md)):
  - **Units:** the opposition's lines, at least two players each.
  - **Line break:** the ball played beyond the deepest player of a unit, through, around or over; the units broken; received inside or outside the shape.
  - **Movement to receive:** in front of the first unit; in between units; in to out or out to in of the shape; in behind the last unit, with sub-types (from the final unit, in front of it, pushing the line, come and go, offside to onside).
  - **Possession states:** in possession, out of possession, in contest, out of play.
  - **Phases:** build-up opposed and unopposed, progression, final third, long ball, counter-attack, counter-press, press against block. FIFA's mid-block test: under 10 km/h, at least 8 outfield players in the middle third, held 1.6 s of 2.0 s.
- **FIFA EFI** metrics computed from tracking: receptions behind the midfield and defensive lines, line height and team length by ball third, final third entries by five channels, ball recovery time, forced turnovers (pressure on the ball, then the ball won on the next touch).
- **Pitch control and value:** Spearman (2018) and Fernández and Bornn (2018). The share of high-value space a team controls is the product of control and a value surface. Plaiq already has both on the honeycomb.
- **Free players and overloads** have no single published definition. Commonly: no opponent within a radius (often 3 to 5 m) or numerical superiority in a zone. Ponta's values are measured, then agreed.
- **Game-state effects:** teams change risk with score and time (for example the trailing team in the last quarter of a match plays more direct). Any risk weight is a judgement until an expert study measures it.

## Objective

Three packages and two smaller additions on the existing registry, each with the full contract: definitions, outputs, Data check entry, tests, golden outputs and versions. Each is measured against the references above so the founder can agree its thresholds and close Gate 3a. Every label without a published basis is named as a Ponta choice.

## Hard constraints

Those of [context-foundations-task.md](context-foundations-task.md) and [context-structure-task.md](context-structure-task.md) carry over:
- additive only;
- the import rule holds;
- deterministic;
- every field defined, versioned, Data-checked and labelled observed or estimated;
- thresholds in the versioned config with a status;
- measure before proposing;
- provider labels never become outputs;
- no names or licensed data in the repo.

In addition:
- **No coordinates and no hex cell indices** in any AI-facing field. Cells are stored for storage, aggregation and the replay; AI-facing fields carry P3 zones, P2 units and player references.
- **Reuse Plaiq's adopted models** (pass completion probability, Choice, Gain version 2, pressure, the threat surface) through `MatchAnalysis`. Do not re-fit them or change their maths. A new score built on them is a new, versioned metric beside them.
- **Agreed outputs do not change silently.** P1 to P4 outputs change only behind a new version with a switch, measured on the 20 SkillCorner matches with a fit and held-out split by match, and adopted only when the founder agrees.
- **Context-adjusted scores never replace a base score.** Base and adjusted are stored side by side; the base stays the default; the adjusted score is labelled estimated until Gate 3b.
- **A runtime budget:** P5 to P7 together within 60 s per match on a laptop, measured on the 20 matches (the Gate 3 budget, to be agreed). Per-frame model calls are subsampled where needed, and the rate is a threshold.

## Deliverables

### 1. P1 0.3.0: in contest from logged events

- Adopt FIFA's four possession states as P1's vocabulary (`in_possession` per team, `in_contest`, `out_of_play`), mapped from the current labels (home, away, contested, dead ball, unknown). The current fields stay, unchanged.
- **In contest from events:** where a logged feed carries duels, clearances, blocks, deflections or interceptions, the ball is in contest from that touch until a player of either team gains control (P1's existing control rule). The tracking trigger (`p1_loose_on_opponent_touch`) stays off.
- **Outcomes:** add "interrupted" (touched by the opponent, no turnover) and the out-of-play variants to the chain outcomes.
- **Measured against:**
  - IDSSE's official possession;
  - the founder's possession replay check (the 14 "nobody" moments);
  - the two phase review cases above (Metrica 1 grid point 9197, Metrica 2 grid point 4206), and SkillCorner grid point 24345 where no feed exists, reported as not fixable without events.
- **Data check:** in contest is Degraded or Blocked without a logged feed, with the reason.

### 2. P5 Relationships, completed (depends on P1, P2, P3, Plaiq)

- **Line breaks, 0.2.0:**
  - **failed passes:** use the logged target where one exists, and Choice's chosen cell for the target point; no inference beyond that;
  - **"over":** only where the provider logs pass height on every pass (StatsBomb, IDSSE). SkillCorner's `high_pass` is logged on completed passes only and must not be used (`PLAIQ_REFERENCES_REPORT.md`, pass height);
  - **units broken**, and received inside or outside the shape (P3's hull).
- **Line-breaking options at the moment of each pass:** from Choice's option cells, the options beyond a unit whose completion probability is at least an agreed value; count, and whether the best one was taken. Measured against SkillCorner's `n_passing_options_line_break` and the passing-option line-break flags.
- **Per grid point, at a measured rate:**
  - Plaiq pressure on the ball carrier (the 1 to 10 scale), and EFI-style pressure on the ball (defenders within a radius, weighted in front);
  - goal-side counts per team, offside line height, and marking distance per defender (nearest attacker);
  - players by P3 zone, overloads by zone, and free players (no opponent within an agreed radius), with "free between the lines" from P3's bands;
  - open passing lanes from the ball carrier: team-mates whose pass completion probability is at least an agreed value.
- **EFI aggregates per team and half:** receptions behind the midfield and defensive lines, final third entries by channel, line height and team length by ball third, ball recovery time, forced turnovers.
- **Measured against:**
  - SkillCorner's line-break flags, `goal_side_start` and `goal_side_end`, `dangerous`, and pressure levels;
  - FIFA's published World Cup 2022 averages, as a sanity check only.
- **Data check:** ball-relative fields Degraded where the ball is undetected; pressure and lanes inherit Plaiq's states; aggregates state their coverage.

### 3. P6 Movement types (depends on P1, P2, P3)

- **A rule-based classifier on FIFA's categories, relative to P2's units and P3's shape:**
  - **in possession:** in front of the first unit, in between units, in to out, out to in, in behind (from the final unit, in front of it, pushing the line, come and go, offside to onside), support, overlap and underlap;
  - **out of possession:** pressing run, recovery run;
  - holding position.
- **Movement episodes** per player, with start, end, type, speed, the units involved and the P3 band at the end, linked to the existing off-ball run rows in the semantic events.
- **Measured against SkillCorner's ten off-ball run subtypes** through a mapping table fixed before scoring. For example `behind` maps to in behind, `coming_short` and `dropping_off` to in front or in between, `overlap` and `underlap` as named, `pulling_wide` and `pulling_half_space` to in to out. Report agreement per type and the confusions, with a fit and held-out split by match.
- **A founder spot check:** 50 replayed movements (as in the phase review), sampled across types.
- **What tracking cannot show:** offers that are only a signal or a body shape (FIFA codes them by hand). Say so in the Data check and never claim them.
- **Data check:** coverage is the share of player time classified; Degraded where a player is unobserved.

### 4. P7 Space, value and game-state context (depends on P1, P4, Plaiq)

- **Per frame and segment, on the honeycomb:**
  - each team's share of high-value space (pitch control times Plaiq's threat surface, summed over cells);
  - the most dangerous space the defending team does not control, as a P3 zone and lane, never a cell index;
  - stored per cell for the replay at a measured rate.
- **A game-state risk weight:** a versioned lookup by goal difference and time band. Its values are a judgement, proposed with a reason and labelled estimated.
- **Context-adjusted Choice, a new metric beside Choice:**
  - `U = p × value − λ × (1 − p) × cost`. Here `p` and `value` are Choice's own parts (pass completion probability and option value at each option's best cell), `cost` is the opponent's threat at the cell where the ball would be lost, and `λ` comes from the game-state weight;
  - the adjusted score is the chosen option's `U` over the best `U`, stored beside Choice with the weight named in the working;
  - **Choice stays the default**; the adjusted score is estimated until the expert study (Gate 3b).
- **Measured against:**
  - SkillCorner's `dangerous` and its passing options' expected threat;
  - the game-state pattern in SkillCorner's events (direct play by goal difference and time), as a check that the weight points the right way.
- **Data check:** adjusted values Degraded or Blocked when game state is unknown; space shares inherit P2's shape coverage.

### 5. P4 view: FIFA's vocabulary beside SkillCorner's

- Keep P4's SkillCorner labels (what Ponta measures against) and publish a **mapped FIFA view** beside them.
- **Add FIFA's splits** that the data supports:
  - build-up opposed and unopposed, from P5's pressure;
  - press against block;
  - counter-press: a regain attempt within seconds of a loss. Measured against SkillCorner's pressing chains.
- **Build FIFA's mid-block rule** (under 10 km/h, at least 8 outfield players in the middle third, held 1.6 s of 2.0 s) and compare it with P4's block from the deepest three defenders.

### 6. Measurement and report

- Every deliverable against its references, with a fit and held-out split by match where anything is tuned. Confusion matrices and κ for labels; correlation and error for quantities.
- **Sensitivity sweeps** for every threshold without a published value: free-player radius, overload rule, lane probability, pressure radius, movement speeds and durations.
- **Runtime and storage** per match and per package, against the 60 s budget.
- **A note on what changed upstream:** none of P1 to P4's agreed outputs, unless a switch was adopted.

## Acceptance criteria

- All packages run on the regression matches (Metrica 1 to 3, SkillCorner 1886347) and on SkillCorner's 20 matches through the registry. They write version-stamped columns and segment tables and attach Data check entries.
- Golden outputs are captured and a second run matches.
- Measurements exist for every item above; every proposed threshold and bar cites them.
- Within the runtime budget, or the overrun is reported with its cause.
- The full local gate passes, existing outputs are unchanged, and the import check and lint pass.
- No player name, licensed data, coordinate array or cell index reaches the repo or any AI-facing field.

## Explicitly not in scope

- P8 semantic states and episodes (Stage D) and the AI layer.
- Analyst studies (Gates 2b and 3b).
- Re-fitting or changing Plaiq's models.
- Expected goals and shot quality (Blocked on SkillCorner).
- FIFA's hand-coded concepts (offers, pass intention, goalkeeper technique).
- Real-time use.
- Ingesting provider labels as outputs.
- Any public FIFA claim.

## Final report (`CONTEXT_RELATIONSHIPS_REPORT.md`, repo root)

Include:
- what was built;
- the measurements against each reference;
- proposed thresholds and the Gate 3a bars with evidence;
- where and why labels disagree with the references;
- the two review cases and the in-contest result;
- runtime and storage against the budget;
- deviations;
- decisions for the founder.
