# Ponta Context Packages

> **Status: adopted 17 November 2026.** Stage A (P0, P1) is committed. Stages B to D (P2 to P8) are planned but not scheduled; P9 belongs to Phase 5. Source: the founder's work plan "Ponta Context Packages: Work Plan" (17 November 2026), adjusted after review. The four founder decisions are recorded below.

See [README.md](README.md) for the document index, [ai-layer.md](ai-layer.md) for how the AI reads what these packages produce, [data-tiers.md](data-tiers.md#context-packages-by-data-type) for what each kind of data can support, and [roadmap.md](roadmap.md) for where the work sits.

## Why this exists

The AI layer can only explain what its context describes, and today that context stops at single actions: one row per pass, carry, run or defensive action, with its score, the pressure on the player and a pitch zone ([ai-layer.md](ai-layer.md#the-semantic-football-event)). That cannot answer the questions coaches actually ask: how a team built up against a high press, when its shape broke, why the opponent's counters hurt. An AI built over single actions would summarise events, and it would sound generic quickly.

Context packages add the missing layers: **possession, game state, team structure, relative zones, phase of play, relationships, movement types, space and value, and match episodes**, each computed deterministically at ingestion, each defined, versioned and covered by the Data check.

## Strategy change (decided 17 November 2026)

**From** "an explainable decision-quality engine with a grounded AI brief", where Decision Quality is the headline and overlaps heavily with SkillCorner Game Intelligence ([competitive-landscape.md](competitive-landscape.md#skillcorner-game-intelligence)).

**To** "a **match context model** that the AI explains": Ponta compiles any supported provider's data into one versioned, Data-checked description of the whole match (possession, phase, shape, relationships, space), and every AI answer cites the moments or episodes behind it. Decision Quality becomes one field among many.

What changes:

- **Differentiation** moves from one metric to the breadth and honesty of the context, and its consistency across providers.
- **The AI build waits for the context.** [ai-workflows-task.md](ai-workflows-task.md) starts after Stage A at the earliest, preferably after Stage B.
- **Wedge A's pitch** becomes "ask anything about your match, every answer cited to an episode" ([go-to-market.md](go-to-market.md#two-wedges-one-to-lead-with-added-15-november-2026)).
- **Discovery calls (E1)** add: which context questions can you not answer today, and do you rebuild provider metrics yourselves?

What does not change: bring your own data, post-match batch only, Tier 1 first, the Data check, the evidence gates E1 to E3, and every claim rule in [ai-layer.md](ai-layer.md#hard-rules-non-negotiable).

### Founder decisions

1. **Repositioning agreed.** The AI build waits for Stage A at the earliest, preferably Stage B.
2. **Ponta computes its own context.** Where a club already has provider-derived metrics (for example a provider's phases of play or run categories), they are stored against Ponta's vocabulary with their source, for comparison and validation, never as Ponta's source.
3. **Unvalidated fields may reach analysts**, labelled "estimated, not yet validated". They never reach a player-facing view until their validation step has passed.
4. **Commit to Stage A only.** Stages B to D are planned and unscheduled; P9 is Phase 5.

## Principles

- **Code computes, the model explains.** Packages are deterministic code run once at ingestion. The model never computes geometry.
- **No coordinates reach the model.** Packages output labelled fields and segments, never raw positions.
- **Every field is defined, versioned and checked:** a written definition, named thresholds, a package version, a Data check state and a coverage figure.
- **Observed or estimated, always labelled.** Every field is tagged as an observed fact (a distance, a count, a direction) or a model estimate (value, control, decision quality, an inferred label).
- **Context can change the objective, transparently.** Game state and phase change what counts as a good action (a safe pass at 3–0 in the 87th minute should not score as a loss), but the base score is always kept, and any adjustment names its weights in the working.
- **Works on any supported provider, honestly.** Packages read Matrix's canonical schema, never one vendor's format; what each kind of data can support is stated per package ([data-tiers.md](data-tiers.md#context-packages-by-data-type)).
- **Measure before setting a threshold.** Thresholds are proposed from measurements on the regression set, then agreed, named and versioned; none is guessed into code.
- **Validated before players see it.** Analysts may see unvalidated fields, labelled as such; player-facing views wait for validation.

## The package contract

A context package is one self-contained module that turns canonical tracking into named context fields. It is done only when all eight items exist.

| Item | What it must contain |
|---|---|
| Definition spec | Plain-language meaning, the formula, every threshold named and versioned (for example `high_line_m = 40`), known limits |
| Inputs | Which canonical fields and which earlier packages it reads |
| Outputs | Field list with units and an observed or estimated tag; written to the context frame table, segment rows, or both |
| Data check entry | Rules for Full, Degraded and Blocked per data type, the reason text, and a coverage line (share of match time or events labelled) |
| Tests | Unit tests, golden outputs on the regression set, and edge cases: ball undetected, half-time, substitutions, red cards |
| AI exposure | Which fields reach `semantic_states` or `semantic_events`, and which filters it adds to the typed query tool |
| Validation plan | How agreement with analysts (or with event data) is checked, and the bar before player-facing use |
| Version stamp | `package_version` on every row it produces |

**Storage.** Frame-level fields go to `context_frames.parquet` on a fixed grid (5 Hz proposed), keyed by `match_id` and `frame_ts`. Segment-level outputs go to `semantic_states.parquet` (built in P8). Both sit beside `trajectories.parquet` and `semantic_events.parquet` under the tenant's prefix and are read through DuckDB, by the API only ([architecture.md](architecture.md#canonical-store-and-ai-layer)).

**Regression set.** Four tracking matches are available locally, not two: Metrica Sample Games 1, 2 and 3 and SkillCorner 1886347. Games 1 and 2 need the known second-half event-flip issue handled first (`to_single_playing_direction`, reported in `ponta-platform/EXTRACTION_REPORT.md`). SkillCorner's open-data repository publishes more matches than the one used so far; count and licence to be checked. Licensed or unlicensed sample data never enters a repository.

## What already exists to build on

- `plaiq_engine/analytics/hex_analytics.py`: `infer_possession` (event-anchored, frame by frame) and `tag_possession`. The baseline P1 is compared against, and its fallback when the ball is undetected.
- `plaiq_engine/statsbomb/stitch.py`: `get_possession_chains` for event-only data.
- `MatchAnalysis.possession` and `MatchAnalysis.pressure(frame)`: pressure for any frame, not only at events.
- `parity.py`: the Data check generator, extended to accept package entries (P0).
- `semantic_builder.py`: once P1 exists, each semantic event also carries its possession chain, phase and game state.

## Build order and gates

```mermaid
flowchart TB
  A["Stage A: P0 Foundations, P1 Possession and game state<br/>(committed)"]
  G1{"Gate 1: possession agrees with event data<br/>above the agreed bar"}
  B["Stage B: P2 Structure, P3 Relative zones, P4 Phase of play<br/>(planned)"]
  G2{"Gate 2: analysts agree with phase and formation labels"}
  C["Stage C: P5 Relationships, P6 Movement types, P7 Space and value<br/>(planned)"]
  G3{"Gate 3: runtime budget met; adjusted decision quality tested"}
  D["Stage D: P8 Semantic states (P9 Embeddings in Phase 5)"]
  A --> G1 --> B --> G2 --> C --> G3 --> D
```

Packages within a stage can run in parallel. **Gate 1 can be met alone**, because the Metrica matches include event data. **Gates 2 and 3 need one or two analysts**, so they depend on discovery calls producing a design partner (Track V in [roadmap.md](roadmap.md)); recruiting them is part of this plan, not a side task.

## Packages

Thresholds marked "agreed" are set in [Open decisions](#open-decisions). "Layer" refers to the founder's context layers.

### P0 Foundations (Stage A, committed)

**Depends on:** Matrix canonical schema.

- This document is the package contract.
- A resampling step from canonical trajectories to the context grid, keeping the source rate for precise refinement.
- The `context_frames.parquet` schema: `match_id`, `frame_ts`, `period`, `package_version`.
- A package registry: name, version, dependencies, run order.
- The Data check generator extended to accept entries from packages.
- A golden-output harness for packages on the regression set.

**Done when:** an empty test package runs end to end on the regression set and writes a stamped frame file and a Data check entry.

**Execution spec for P0 and P1:** [context-foundations-task.md](context-foundations-task.md) (ready, 17 November 2026).

### P1 Possession and game state (Stage A, committed; layer 1)

**Depends on:** P0; event data where available.

- Possession states: home, away, contested, dead ball.
- Possession from ball proximity and control when the ball is detected (observed).
- Possession during ball-detection gaps inferred from player movement, tagged inferred; the event-anchored baseline is the fallback and the comparison.
- Possession chains: start, end, how each began and ended.
- Game state on every frame: score, minute, red cards, dead ball, set-piece type.
- An upload step for final score, goal times and red cards when there is no event data.
- Data check: possession Degraded when the inferred share passes the agreed threshold; game state Blocked when unknown.
- **Measure first:** ball-undetected share, inferred-possession share and agreement with event data on every match in the regression set, reported before any threshold is agreed.

**Done when:** every frame carries a possession label and its source (observed or inferred), and possession agrees with event data on Metrica Game 3 above the agreed bar (Gate 1).

### P2 Team structure (Stage B, planned; layer 5)

**Depends on:** P1.

- Each team's defensive, midfield and attacking lines, in and out of possession.
- Line heights (metres from own goal), distance between lines, width, depth and compactness (observed).
- Formation per phase, smoothed over time windows (for example 4-4-2 out of possession, 2-3-5 in possession) (estimated).
- A role per player per window, including variants such as an inverted full-back (estimated).
- Shape changes against the previous window.
- Data check: Degraded when players observed fall below the agreed threshold.

**Done when:** formation labels match analyst labels on sampled segments from the regression set (Gate 2).

### P3 Relative zones (Stage B, planned; layer 2)

**Depends on:** P2.

- Relative zones: between the lines, behind the back line, the pocket beside the 6, wide of the full-back, half-space between lines.
- The ball and each player labelled per frame with a fixed zone and a relative zone.
- Data check: inherits P2's state.

**Done when:** zone labels pass a visual spot check against the replay on 50 sampled frames.

### P4 Phase of play (Stage B, planned; layer 7)

**Depends on:** P1, P2, Plaiq pressure.

- Phase taxonomy: build-up, progression, final third, settled possession; high press, mid block, low block; attacking transition, counter-press, defensive transition; set pieces.
- Labelling rules from possession, ball location, block height, pressure and time since turnover.
- Segmentation into phases with a minimum duration and smoothing.
- Data check: coverage as share of match time labelled; Degraded when possession was inferred.

**Done when:** phase labels agree with two analysts on sampled segments above the agreed bar (Gate 2).

### P5 Continuous relationships (Stage C, planned; layer 4)

**Depends on:** P1, P3, Plaiq.

- Plaiq pressure on the context grid, not only at events.
- Goal-side counts per team, offside line height, marking distance per defender.
- Players by zone; overloads and free players (no opponent within the agreed distance), especially between the lines.
- Open passing lanes from the ball carrier, reusing pass option logic at lower cost.
- Data check: ball-relative fields Degraded when the ball is undetected.

**Done when:** all fields are computed for every frame of the regression set within the agreed runtime budget (Gate 3).

### P6 Movement taxonomy (Stage C, planned; layer 3)

**Depends on:** P1, P2, P3.

- Movement types: run in behind, checking towards the ball, run across the line, support run, recovery run, pressing run, holding position.
- A rule-based classifier from speed, direction and position relative to goal, ball and lines.
- Movement episodes per player, linked to the existing off-ball run rows.
- Data check: coverage as share of player time classified.

**Done when:** labels agree with analysts on a sampled set of runs above the agreed bar.

### P7 Space, value and game-state context (Stage C, planned; layer 6)

**Depends on:** P1, P4, Plaiq.

- Control and value per frame and per segment: each team's share of high-value space, the most dangerous uncontrolled zone.
- A game-state risk weight as a versioned lookup by goal difference and time band.
- Context-aware utility and context-adjusted decision quality: `U = p × gain − λ × (1 − p) × cost of losing the ball`.
- Base and adjusted values stored side by side; the working names the weight and why. **Base stays the default**; the adjusted score is labelled an estimate until the expert study measures it, because the weights are a judgement, not a measurement. Adding the adjusted score is a new metric with its own version, not a change to the existing formula.
- Data check: adjusted values Degraded or Blocked when game state is unknown.

**Done when:** the expert study measures analyst agreement with adjusted decision quality against base decision quality (Gate 3).

### P8 Semantic states and episodes (Stage D, planned; layer 8)

**Depends on:** P1 to P7.

- The `semantic_states` schema: `state_id`, `match_id`, `start_s`, `end_s`, phase, possession, shapes, line metrics, relationships, space summary, Data check states, package versions.
- A segmenter that merges frames into episodes on phase or shape change, with a minimum duration.
- A short templated text summary per state (no model).
- The citation verifier extended to accept `[state_id, start_s, end_s]`.
- State filters and episode playlists in Ask Ponta's typed query tool.
- Golden and adversarial questions about states in CI.

**Done when:** Ask Ponta answers state questions with cited episodes on the regression set and every guardrail test passes.

### P9 Embeddings (Phase 5; layer 9)

**Depends on:** P8 and the agreed minimum number of matches, which needs design-partner data.

- A hand-built feature vector from P2 to P7 first; a learned model later, if at all.
- Fixed windows and states embedded, stored against `state_id`.
- A similarity tool: "find moments like this".
- Evaluated with pairs analysts rate as similar or not.

**Done when:** analysts rate the top five results as similar above the agreed rate.

## Cross-cutting work

- **Vocabulary:** one versioned context vocabulary (every term, its definition, thresholds and the package that produces it), plus a mapping table so provider-derived metrics can be stored against it with their source (decision 2).
- **Data check and coverage:** one Data check family per package, with coverage beside each state; the onboarding view shows what a second source or event data would unlock.
- **AI layer:** the typed query tool's closed filter schema grows with each package, never ahead of it; the citation verifier covers state citations and quoted context numbers; golden and adversarial questions exist for each package before its fields reach the model; every field shown to users is labelled observed or estimated, and unvalidated ones "estimated, not yet validated" (decision 3).
- **Validation:** the planned expert-agreement study ([engine-governance.md](engine-governance.md#construct-validity-and-reliability-of-decision-quality-added-15-november-2026)) widens to possession, phase, formation, movement types and adjusted decision quality; a small labelling tool on the replay lets analysts label segments and runs quickly.
- **Test data and performance:** the regression set above; processing time and storage per match measured after each package against the agreed budget.
- **Player-facing readiness (later):** which context fields may feed player pattern cards, and the minimum sample per pattern.

## How this fits the roadmap

- **The concierge pilot and gate E2 go first.** The founder-run match page does not wait for any package; P0 and P1 can feed it once ready, because possession chains and game state make its moments clearer.
- **Stage A is engine work** in `ponta-platform`, like the extraction, so it is not gated by E1 and can start now.
- **Stages B and C sit inside Phase 2**, extending the Tier 1 pipeline; P8 completes Phase 2's AI work, and Ask Ponta's exit criteria gain state questions with cited episodes.
- **P9 and player-facing patterns belong in Phase 5.**

## Open decisions

Each is decided before its package starts, from measurements where the table says so.

| Decision | Needed before | Notes |
|---|---|---|
| Context grid rate (5 Hz proposed) | P0 | Lower is cheaper; refinement uses the source rate |
| Inferred-possession share that makes possession Degraded | P1 | Measured on the regression set first |
| Possession agreement bar with event data | Gate 1 | Metrica Games 1 to 3 have event data to compare against |
| Minimum players observed for structure to be Full | P2 | The existing optical rule is 10% of player observations missing |
| Phase taxonomy: final list and minimum phase duration | P4 | Agree with one or two analysts before coding |
| Agreement bar for analyst studies | Gate 2 | Same method as the decision-quality study |
| Free-player distance and overload rules | P5 | Named and versioned like the high-pressure threshold |
| Runtime and storage budget per match | Gate 3 | Sets how much of P5 runs on the full grid |
| Game-state risk weights by goal difference and time band | P7 | A simple lookup first, shown in the working |
| Minimum number of matches before embeddings | P9 | Depends on design-partner data |
| Publish the vocabulary or keep it private | Cross-cutting | Same question as the analytics specification |

## What this document does not do

- It does not schedule Stages B to D or set dates; each stage starts when the gate above it is met.
- It does not change the existing engine formulas; new fields and the adjusted score are new, versioned outputs.
- It does not choose an LLM provider or design the UI for context fields.
- It does not claim any package's labels are accurate before its validation step has run.
