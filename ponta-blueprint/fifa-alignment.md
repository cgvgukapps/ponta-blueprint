# Alignment with the FIFA Football Language

> **Status: analysis, 18 November 2026.** How Ponta's context model compares with FIFA's published Football Language and its Enhanced Football Intelligence (EFI) metrics, and what to change. Internal and candid: it separates what is built, what is planned and what is missing. It is not a claim of FIFA endorsement, and nothing here may be used publicly before the [usage note](#using-fifas-name-and-material) is cleared.

## Sources

- **The Football Language** (FIFA Training Centre, 2021 to 2022; 117 term pages). Five sections: In possession, Out of possession, In contest, Goalkeeping, Possession outcomes. Starting points:
  - [the Football Language index](https://www.fifatrainingcentre.com/en/resources-tools/football-language/index.php);
  - [the article "The FIFA Football Language"](https://www.fifatrainingcentre.com/en/game/performance-analysis/football-language-analysis/the-fifa-football-language.php).
- **EFI explanation documents** v1.0 (World Cup 2022) and v1.1 (Women's World Cup 2023). They define the 11 EFI metrics, including the phases of play.
- **FIFA's World Cup 2022 article** on the mid-block, the only source with numeric phase thresholds.
- Read in full on 18 November 2026; the extracted text is kept outside the repos. Videos and diagrams on the term pages were not extracted.

## The short answer

- **Ponta and FIFA build the game the same way.** Both describe it relative to the opposition's units (lines) and shape, from player-tracking positions, with phases computed by rules on tracking. FIFA's EFI phases are "based on tracking data", which is Ponta's P4 approach.
- **Several FIFA measures are already built in Ponta with the same definition** (see [What Ponta already matches](#what-ponta-already-matches)):
  - defensive line height and team length;
  - the five vertical channels;
  - "between the units" receiving locations;
  - possession share by state.
- **The main differences:**
  - **Vocabulary:** P4 uses SkillCorner's phase names, chosen so phases could be measured against a reference. FIFA's names differ and split some phases (build-up opposed and unopposed, press against block).
  - **"In contest":** FIFA treats it as a possession state in its own right. Ponta's P1 has only "contested" (both teams near the ball). The founder's loose-ball rule is FIFA's in-contest, but Ponta's tracking trigger is not yet good enough ([below](#possession-states)).
  - **What FIFA codes by hand,** such as offers to receive, pass intention, crosses and goalkeeper techniques, Ponta cannot see in tracking without a feed.
  - **Line breaks** (FIFA's flagship metric) are not built in Ponta yet; the parts exist.
- **Ponta cannot measure agreement with FIFA's labels,** because FIFA publishes no labelled data. Alignment here is by definition, not by measured agreement.

## Possession states

| FIFA | Definition (paraphrased) | Ponta (P1) | Alignment |
|---|---|---|---|
| In possession / out of possession | A player of the team has, or an opponent has, **controlled** possession | `home` / `away` | **Aligned in principle.** P1 calls control from distance (1.5 m) and ball speed, held 0.4 s to switch |
| **In contest** | "When neither team has controlled possession of the ball": loose balls after duels, clearances, rebounds, blocks, deflections, misplaced passes | `contested` (both teams within 1.5 m) only. `loose` exists behind `p1_loose_on_opponent_touch`, off | **Partly.** P1 keeps the last team through most in-contest moments |
| Out of play | Ball not in play | `dead_ball` (from a logged feed; unknown without one) | Aligned where a feed exists |
| EFI "Possession control" | Share of time: Team A, Team B, In contest, Out of play, "calculated based on the sequence of events" | P1 label shares | Same four-way split once in-contest is fixed. **FIFA's is event-based**; Ponta's is tracking-based |
| EFI "Ball recovery time" | From a team's last ball-control event in one possession to its first in the next | Not built; computable from P1 chains | Easy to add |

**What FIFA's approach suggests.**
- The founder's rule (a deflection or clearance leaves the ball in contest) is FIFA's definition.
- Ponta's tracking trigger over-fires: on the founder's judged phase moments it cost 2 of 50, and two more matches fell below the phase floor (`CONTEXT_STRUCTURE_REPORT.md`, possession round).
- FIFA derives possession states from events. **With a logged feed, take "in contest" from the logged clearances, blocks, duels and misplaced passes.** Without a feed, keep the rule off until a better touch detector is measured.

## Phases of play

FIFA's EFI phases (v1.1) are built from tracking, frame by frame, and kept when the same phase holds for "a sufficiently large number of consecutive frames". Ponta's P4 uses SkillCorner's published vocabulary.

| FIFA phase | FIFA's definition (paraphrased) | Nearest Ponta P4 label | Gap |
|---|---|---|---|
| Build up unopposed / opposed | Starting the attack from the back, short passes; unopposed under minimal pressure, opposed when the opponent engages | `build_up` | Ponta has no opposed / unopposed split. Plaiq pressure could provide it |
| Progression | Advancing the ball into the final third (line-breaking passes, carries) | `create` (closest) | Different cut: Ponta's create follows the defending block |
| Final third | In possession in the attacking third, aiming to finish | `finish` | Ponta's finish needs a low block and the ball in the attacking half; FIFA's is positional (the final third) |
| Long ball | Not defined in FIFA's published text | `direct` | Comparable by name only |
| Counter-attack | Regain and attack immediately "with speed and intensity" | `quick_break` / `transition` | On IDSSE, Ponta's breaks catch about a third of the league's counter-attack flags on shots |
| Attacking transition | Named, not defined | `transition` | — |
| Set play: corner, free kick, **throw-in**, penalty | FIFA counts throw-ins as set plays | `set_play`: corners and attacking free kicks only (SkillCorner's convention); throw-ins are restarts | Vocabulary choice; Ponta now detects throw-ins (P4 0.3.0) |
| High press / high block | Engaging high and aggressively (press); organised shape (block) | `high_block` | Ponta has no press against block split. Plaiq pressure plus speeds could provide it |
| Mid press / **mid-block** | Mid-block published with thresholds (below) | `medium_block` | Ponta's block comes from the deepest three defenders' depth (26 m / 47 m), not FIFA's rule |
| Low press / low block | Organised shape in the defensive third | `low_block` | As above |
| Counter-press | Immediately trying to win the ball back after losing it | `defending_transition` / `defending_quick_break` (partly) | No counter-press label |
| Recovery | Running back towards goal after losing the ball | (none) | Not built |
| Defensive transition | Named, not defined | `defending_transition` | — |
| (no FIFA phase; in contest is a state) | — | `chaotic` | Ponta's chaotic sits closest to FIFA's in-contest state |

**FIFA's published mid-block rule** (World Cup 2022 article), the only phase with numbers:
- **Stationary:** no player moving forward or backward faster than 10 km/h.
- **Organised:** the players (except the highest) form horizontal lines, the team has been stationary for at least 1.6 s of the previous 2.0 s, and at least four defenders are behind the ball.
- **Area:** at least eight outfield players in the middle third, or the middle and defensive thirds. Eight in the defensive third alone is a low block.
- **End:** the criteria fail for 0.4 s within 2.0 s, possession changes, or the ball goes out.
- **For Ponta:** all of this is computable from tracking today (P2 lines, speeds, P1 possession). Building it beside Ponta's block would show whether the two agree.

## Units, lines and where the ball is received

| FIFA | Ponta | Alignment |
|---|---|---|
| **Units**: the opposition's lines. A unit line has at least two players; EFI detects units from tracking ("groups of players that act in similar roles") | P2 lines: defence, midfield and attack by exact 3-means on depth, per two-minute window | **Aligned in concept.** FIFA allows up to four units, Ponta three, and Ponta does not require two players per line |
| **Team shape**: formed by joining the widest players of each unit; actions inside or outside it | P3 hull: convex hull of the defending outfield players; ball inside or outside | Close. FIFA's shape is unit-based; Ponta's is a hull |
| **Receiving location**: in front of unit 1, between units 1-2, 2-3, 3-4, in behind; inside or outside the shape | P3 band: `in_front`, `between_mid_att`, `between_lines`, `in_defensive_line`, `behind_defence`; pocket | **Aligned**, for the ball's location at every frame |
| **Line break**: ball played beyond the deepest player of a unit; through, around or over; units broken; received inside or outside | Not built (P5 plans open passing lanes; Plaiq has pass options) | **Missing.** The parts exist: P2 units, P3 bands, passes in the semantic events |
| **EFI receptions behind midfield and defensive lines** | P3 band at receptions | Computable now |
| **EFI defensive line height** (deepest defender to own goal) and **team length** (deepest to highest outfield player, no goalkeeper), by possession and ball third | P2 `def_line_m` and `length_m` per frame, with possession and ball third from P1 and P3 | **Same definitions** |
| **Compactness** (width × length in a block); World Cup 2022 mid-block average 40.1 m × 27.2 m | P2 `width_m` × `length_m` | Same measure. A comparison needs FIFA's block rule |
| **EFI final third entries**, by five vertical channels (left, left inside, central, right inside, right) | P3 lanes: `left_wing`, `left_half_space`, `centre`, `right_half_space`, `right_wing` | **Five channels each.** FIFA does not publish its channel edges; Ponta uses the penalty-box and six-yard-box edges |
| **EFI team shape**: functional roles from positions relative to team-mates, aggregated over time | P2 formations and roles (template fit, main role from the defensive shape) | Similar intent; different method |

## Pressure

| FIFA | Ponta | Alignment |
|---|---|---|
| Coded pressure on receiving, passing, crossing and shooting: **direct** (actively contesting, or body contact), **indirect** (controlling direction, not close enough to compete), **none**. No distances or times are published | Plaiq 1v1 pressure on a 1 to 10 scale, from distance and angle; no named threshold yet | Different form. Ponta's open decision on a named high-pressure threshold could take FIFA's three bands, calibrated on a judged sample |
| EFI pressure on the ball: from tracking, using defender distance, angle (in front counts more than behind) and the number of defenders nearby; "moderate" or "heavy"; no thresholds published | Plaiq pressure | **Close in method**, both tracking-based on distance and angle |
| EFI forced turnovers: pressure on the ball, then the team wins it on the next touch | Not built; computable from Plaiq pressure and P1 chain starts (`won`) | Easy to add |

## Movement and offers

- **FIFA codes by hand** "offering to receive" (a deliberate signal, body shape or movement) and "movement to receive":
  - in front of the first unit;
  - in to out, or out to in, of the shape;
  - in between units;
  - in behind the last unit, with sub-types: from the final unit, in front of it, pushing the line, come and go, offside to onside.
- **Ponta's planned P6 movement taxonomy** (Stage C, not built): run in behind, checking towards the ball, run across the line, support run, recovery run, pressing run, holding position.
- **Recommendation:** define P6's in-possession movements on FIFA's categories, relative to P2's units and P3's shape, which is how FIFA defines them. Offers that are only signals or body shape are not visible in tracking; say so in the Data check.

## Possession outcomes and events

| FIFA | Ponta | Alignment |
|---|---|---|
| Possession outcomes: complete, incomplete, **interrupted** (touched by the opponent, no turnover), lost, won, retained, out of play won / lost / interrupted, goal, own goal | P1 chain start types (`won`, `recovery`, `set_piece:*`, `period_start`) and end types (`lost`, `dead_ball`, `shot`, `goal`, `period_end`) | Partly. FIFA's outcomes are per action; Ponta's are per possession. Ponta has no "interrupted" (the in-contest gap again) |
| Possession actions (assist, blocked, cleared, deflected, foul for / against, intercepted, miscontrol, misplaced, offside, tackled …) | From the provider's events, where they exist | Depends on the feed |
| Defensive events (block, clearance, interception, tackle, take-on against), duels (aerial, physical, ground; won is first contact) | From the provider's events; not derived from tracking | Depends on the feed |
| Goalkeeping (goal prevention, aerial control, defensive line support, active engagement, distribution style) | Not modelled | Out of scope for now |
| EFI expected goals (distance, angle, obstruction, pressure, body part, keeper position) | Not built (shot quality is Blocked on SkillCorner) | Missing |

## Method: how FIFA's data is made, and how Ponta's is

- **FIFA's EFI** combines:
  - **manual event coding by trained analysts**, with intention, offers and techniques;
  - **automated enrichment from tracking:** units, line breaks, phases of play, line heights and pressure on the ball.
- **Ponta** computes from the club's own provider data: tracking first, with the provider's events where they exist, and no manual coding.
- **What Ponta shares with FIFA** is the automated half:
  - **in the same way:** units from tracking, phases by rules on tracking, line heights;
  - **differently:** pressure on the ball and possession states. FIFA takes possession states from events; Ponta's pressure is its own 1 to 10 scale, although FIFA's EFI pressure on the ball works from the same inputs (distance and angle).
- **What Ponta lacks** is the manual half. Ponta should not claim FIFA's coded concepts (offers, pass intention, goalkeeper technique) without a feed that carries them.

## What Ponta already matches

Built, with the same definition as FIFA's:
1. **Defensive line height** (deepest defender to own goal line) and **team length** (deepest to highest outfield player, goalkeeper excluded), each by possession state.
2. **Team width and compactness** (width × length).
3. **Five vertical channels** across the pitch.
4. **Receiving location relative to the opposition's units** (in front, between units, in behind), and inside or outside the shape.
5. **Possession share** by team and stoppage (in contest still partial).
6. **Phases computed by rules on tracking**, with low, mid and high blocks named the same way.

## Recommendations, in order

1. **Possession states:** adopt FIFA's four states (in possession, out of possession, in contest, out of play) as P1's vocabulary. Take "in contest" from logged events where a feed exists; keep the tracking trigger off until a better touch detector is measured.
2. **Line breaks** in Stage C (P5): ball played beyond the deepest player of a unit of at least two players; through, around or over; units broken; inside or outside. FIFA's flagship metric, and the parts are built.
3. **A FIFA-vocabulary view of phases:** keep P4's SkillCorner labels (they are what Ponta measures against) and publish a mapped FIFA view beside them. Add the splits FIFA makes from the data Ponta already has: build-up opposed and unopposed, press against block, counter-press. Build FIFA's published mid-block rule and compare it with Ponta's block.
4. **P6 movement taxonomy on FIFA's categories** (in front, in between, in to out, out to in, in behind with sub-types), relative to P2's units.
5. **Pressure bands:** calibrate direct, indirect and no pressure on Plaiq's 1 to 10 scale with a judged sample. This also settles the open "named high-pressure threshold" decision.
6. **EFI metrics Ponta can compute now:** ball recovery time, forced turnovers, receptions behind the lines, final third entries by channel, line height and team length by ball third. Expected goals stays missing until shot quality is unblocked.
7. **Outcomes:** add "interrupted" and the out-of-play variants to P1's chain outcomes once in-contest exists.

Each item follows Ponta's rules: thresholds measured before they are agreed, labels estimated until analysts validate them, no claims beyond the evidence.

## Using FIFA's name and material

- **The FIFA Training Centre's terms of service:**
  - all text, data and video is FIFA's or its licensors' (§5.1);
  - use is limited to private, non-commercial use (§5.3), and FIFA content may not be used commercially, nor FIFA's names or marks, without prior written agreement (§6.2, §6.5, §7);
  - the landing page says the language "will be available open-source", but no licence or reuse terms are published.
- **Internally:** using FIFA's published concepts to shape Ponta's own definitions is the point of this document. **Do not copy FIFA's text** into the product, the docs shown to clients or the AI's prompts; write Ponta's own definitions.
- **Publicly:** do not say "FIFA-aligned", "FIFA standard" or similar, or use FIFA's name or marks, without legal advice and, most likely, FIFA's written consent. A factual statement such as "phase definitions follow published industry frameworks" is the safe form until then. Any public mention of FIFA needs its own check before launch, alongside the legal-pages gate G4 in [website-v2-plan.md](website-v2-plan.md#launch-gates-public-announcement-not-the-friends-only-url); the position is recorded in [legal-compliance.md](legal-compliance.md#open-reference-datasets-added-18-november-2026).
