# CLAUDE.md — Ponta blueprint

This repo holds no code. It is the **planning and design blueprint** for Ponta, a multi-tenant, AI-native B2B SaaS for football clubs. It covers architecture, scope, sequencing, risk, go-to-market, legal, brand and execution-task specs for coding agents. There is no build, lint or test step here.

## Layout

- Git root: `~/PontaPlan` (remote `cgvgukapps/ponta-blueprint`). All documents live in `ponta-blueprint/`.
- `ponta-blueprint/*.md`: 30 flat Markdown documents that link to each other with relative links and `#anchor`s.
- `ponta-blueprint/assets/`: logos, icons, the PontaFlow diagram and a reference image. Binary; do not edit.
- `.cursor/` is git-ignored and holds only an empty `mcp.json`. There are no Cursor rules to follow.

## Read first

1. `ponta-blueprint/status.md`: the **living handoff note** (state, repo positions, ordered next steps, open decisions, gotchas). Read it at the start of every session and **update it at the end** of any session that changes state.
2. `ponta-blueprint/README.md`: strategy summary, baseline assumptions, the document index, the deferred list.
3. Then whichever doc the task needs (see the index below).

## Document index (by role)

- **Plan:** `roadmap.md` (phases with exit criteria, Track V validation, Track L legal), `data-tiers.md` (Tier 1/2/3, the Parity Report, context by data type), `ai-layer.md` (grounded AI design), `context-packages.md` (context packages P0 to P9; Stages A and B built, decision 4a).
- **Engines and execution tasks:** `Matrix.md`; `context-foundations-task.md` (complete: Stage A built, Gate 1 passed); `context-structure-task.md` (Stage B built; Gate 2a bars agreed and met on the reference match, a role-fit revision pending); `ai-workflows-task.md` (not startable until context Stage A and the API skeleton exist); `platform-extraction-task.md`, `matrix-prototype-task.md`, `matrix-statsports-adapter-task.md`, `prototype-cleanup-task.md` (complete).
- **System design:** `architecture.md`, `engine-governance.md`, `operations.md`.
- **Product and UX:** `features.md`, `frontend.md`, `frontend-design-brief-v2.md` (the fixed UI decisions live in its section 3; v1 superseded), `visual-design.md` (website-first), `website.md`, `website-v2-plan.md` (executed; launch gates open).
- **Strategy and risk:** `risks.md`, `competitive-landscape.md`, `go-to-market.md`, `legal-compliance.md`, `roadmap_v2.md` (investor vision, not a commitment).
- **For outside review:** `platform-overview.md` (a dated, self-contained snapshot of the platform and its evidence status; summarises, does not supersede).

## The product in one paragraph

Ponta is **one application**. **Matrix** is its data engine (Normalise → Reconcile → Fuse provider files into one quality-scored canonical dataset). **Plaiq's** deterministic spatial maths (pressure, control, pass option value, decision quality) runs once at ingestion and is compiled into **semantic football events**. **Context packages** (possession, game state, structure, phase, relationships, space, episodes) describe the whole match, so Ponta is a **match context model that the AI explains**. Grounded AI agents read those events and episodes, cite every claim and read a per-match **Parity Report** (Full / Degraded / Blocked) first. Bring your own data, post-match batch only. Planned stack: Python engines in a `ponta-platform` monorepo, FastAPI, Supabase Postgres, Parquet in object storage via DuckDB, and a web app (Next.js in the docs, Vite in the UI prototype, undecided).

## Related repos (outside this one; read-only unless the task says otherwise)

| Repo | Path | Role |
|---|---|---|
| Prototype | `~/PONTA-PROTOTYPE` | Frozen validation record (tag `prototype-validation-final`). **Never modify.** |
| Platform | `~/Ponta-Platform` | Production monorepo: both engines, the Data check and the semantic builder; `apps/api`, `apps/worker`, `apps/web` are still empty. |
| UI prototype | `~/Ponta_Platform_UI` | Vite/React mock-data prototype (tag `prototype-v0.1`); design decisions in its `docs/design-notes.md`. |
| Website | `~/Ponta-Website` | Marketing site, live but `noindex`. Has its own `AGENTS.md`. |

`status.md` holds the exact commit positions and resume commands for these.

## Rules for editing the blueprint

- **Evidence beats strategy.** Where the new strategy conflicts with an evidence-based limit, the evidence wins unless a recorded decision changes it. Keep validated, synthetic-only and not-built claims clearly separated (see the README's "What is validated vs. not").
- **Never overstate.** Do not call Matrix output "ground truth". Do not claim validation, accuracy or results that have not been produced. The 24.5% RMSE figure is **synthetic only**.
- **Internal docs are evidence-first; public copy is not.** `website.md`'s "promote, don't qualify" tone applies to the public site only. `Matrix.md`, `competitive-landscape.md` and `engine-governance.md` stay candid about limits.
- **Revise by superseding, not erasing.** Docs carry "Revised for the AI-native one-product strategy" banners and mark older sections *(pre-pivot, superseded)*. Follow that pattern so history stays readable.
- **Phases and exit criteria, not calendar dates** in plans. Use absolute dates (for example "15 November 2026") in status notes.
- **Keep cross-links working.** If you rename a heading, fix every `file.md#anchor` that points to it (grep for it).
- **Do not rename or move documents.** Other repos link to them by name: the website repo to `visual-design.md` and `website-v2-plan.md`, the frozen prototype to the Matrix task specs, the platform and UI repos to the task specs and the design brief. Mark a finished or superseded document with a status banner instead.
- **Status banners.** Every execution-task spec and plan carries a dated `> **Status: …**` line under its title (complete, next, executed, superseded). Update it when the state changes.
- **Known contradictions** are listed in `status.md` ("Docs that now disagree with the evidence"). Reconcile them when you touch those docs, and remove items from the list once they are fixed.
- **`platform-overview.md` is a dated snapshot.** Refresh it, or note that it is out of date, when the state it describes changes materially.
- **Execution-task specs** follow a fixed shape: Context / Preconditions, Objective, Constraints (or Hard constraints), Steps or Deliverables, Acceptance criteria, Explicitly not in scope, Final report. Each spec is self-contained for an agent with no other context.

## Writing style

- **British English** (normalise, pseudonymised, colour).
- Plain, specific and confident. No hype words, no exclamation marks, no emoji. Product copy uses no em dashes.
- Use bold for the key term or decision, not for decoration.

## Product hard rules (carry these into any spec, brief or copy you write)

- No raw coordinates in any LLM context. Every AI claim cites `[event_id, timestamp_s, confidence_score]` plus `match_id` (what fills the confidence slot for single-source matches is still open). Parity first. No arbitrary SQL. Pseudonymised prompts.
- Engine scales: decision quality runs from 0 to 1.5 (bands Poor below 0.55, Suboptimal 0.55 to 0.79, Good 0.80 to 0.94, Optimal 0.95 and above); pressure is 1v1 pressure on a 1 to 10 scale with no named threshold yet. Never present either as 0 to 1.
- No medical, injury or causal-fatigue claims. Association only ("coincided with", never "caused by").
- No real player names in fixtures, mocks or generated artefacts.
- Batch only. No real-time or live claims.
- The only real providers are Metrica, SkillCorner, StatsBomb and STATSports. Others are "Coming soon".
- Do not change engine maths during extraction. Report discrepancies; do not "fix" formulas.
- Context packages: code computes, the model explains; every field defined, versioned, Data-checked and labelled observed or estimated; thresholds measured before they are agreed; unvalidated fields reach analysts only as "estimated, not yet validated", never a player-facing view.
- Licensed or unlicensed sample data (Metrica, SkillCorner, STATSports) never gets copied into a repo.

## Git

- Small, descriptive commits on `main` are the norm here (it is a docs repo with a solo author). Push only when asked.
- Do not commit `.cursor/`, `.env*` or `.DS_Store` (already git-ignored).
