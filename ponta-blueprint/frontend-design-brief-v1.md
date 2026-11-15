# Ponta Platform: Frontend Design Brief (v1)

**Version:** v1, for the first round of prototype experiments with an external agent. Expect to revise it after the first prototype is reviewed (see [Review checklist for v2](#review-checklist-for-v2-founder-use-not-part-of-the-prompt)).

**How to use this file.** Everything from "Your task" down to "Report back" is the prompt for the external agent. The final section is for the founder only. Source-of-truth documents are listed at the end; where this brief and those documents disagree, this brief wins for the prototype, and the disagreement should be reported.

---

## Your task

Design and build a **clickable, high-fidelity frontend prototype** of the **Ponta platform**: the authenticated, dark-themed web application where football analysts and coaching staff upload match data, see what that data can and cannot support, read a cited AI match brief, replay the match on a pitch, and ask questions in natural language.

This is a **new design workstream**. Ponta also has a public marketing website with a light, white-bodied design. The platform must *not* look like that site. It shares a brand (mark, accent blue, typeface) but is its own product, with its own dark look. Treat the website as a distant cousin, not a template.

The prototype is for design review. It uses **mock data only**: there is no backend, no real authentication and no real AI. It must look and behave as if the product were real.

## 1. What Ponta is

Ponta is an **AI-native football intelligence product**. A club uploads data from the providers it already uses: optical tracking, GPS or wearable load data, or event feeds. Ponta does three things:

1. **Normalises, reconciles and fuses** whatever it is given into one canonical, quality-scored dataset. This is the built-in data engine, called **Matrix**.
2. **Produces semantic football events** (passes, carries, off-ball runs, defensive actions) and scores the **quality of decisions** players make, using spatial analysis. This is the decision engine, called **Plaiq**.
3. **Lets grounded AI read those events** to produce a match brief and answer questions. Every statement the AI makes links to the exact moment it came from.

Short positioning line (use it as the product's tone, not necessarily as visible copy): *Tactical and physical AI that sees the whole pitch, and the athlete inside it. Every insight links to the moment it came from.*

**Inside the app, Matrix and Plaiq are not separate products.** The user sees one product, "Ponta". The names may appear only in technical detail panels (for example "Processed by Matrix, engine version 0.x"). Do not build separate "Matrix" and "Plaiq" areas.

### The idea that drives the whole interface: honesty about data

What Ponta can say depends on the data a club brings (its **data tier**). Ponta is designed to *tell the user what it cannot do*, not to hide it.

| Tier | What the club provides | What it unlocks |
|---|---|---|
| **Tier 1** | One source (one export) | Cited brief, replay, natural-language explorer, quality report |
| **Tier 2** | Two unpaired sources (for example old and new GPS during a vendor switch) | Workload and transition monitoring, with calibration only where the data overlaps |
| **Tier 3** | Paired sources for the same match (for example GPS + optical) | Fitness-linked decision analysis, shown as association only, never causation |

For every match, the platform stores a **Parity Report**: for each metric family, the state is **Full**, **Degraded** or **Blocked**, with a plain-language reason. Examples: a GPS-only match has opposition pressure and control zones **Blocked**; event-only data gives decision quality **Degraded**. This report is shown everywhere a match appears. It is the product's central trust feature, and the interface should make it feel like a strength, not a warning label.

## 2. Who uses it, and the market

**Market.** Professional and semi-professional football clubs, academies, federations, performance consultancies, recruitment agencies and independent scouts. The first customers are the underserved segment: clubs below the top one or two divisions in major markets, top divisions in smaller markets, and clubs already running lower-cost tracking systems. They have real data but no data-science team and no six-figure analytics contract. Football is a closed, trust-based, relationship-driven industry. The audience is football-literate and sceptical of "black box AI ratings".

**Users (the interface is designed for these people, in this order):**

1. **Performance or video analyst (primary).** Uploads data, checks quality, builds the brief, explores the match. Spends hours in the app. Wants density, speed and precision.
2. **Head coach and coaching staff.** Reads the one-page brief, clicks citations to see the moment on the pitch. Wants clarity in under two minutes.
3. **Sports scientist and medical staff.** Reads load and physical context. Needs careful wording; the product makes no medical claims.
4. **Scout and recruitment analyst** (later). Compares players across matches. Not in this prototype's core scope.
5. **Director of football, federation and academy leads** (later). Consume shared or printed reports.

**What they decide with it:** how the team played, which decisions were good or poor and why, how reliable the data behind a statement is, and whether a second data source is worth bringing in.

## 3. Design direction

**Feel:** modern, precise, serious infrastructure for people who care about numbers. Think calm, dense, high-trust. Reference points for *attitude* (not to copy): developer and analytics tools with careful typography and hairline structure (Linear, Vercel dashboards, trading and telemetry terminals). Not a gaming UI, not a sports-betting UI, not a consumer fitness app.

**Principles:**

- **Precision over decoration.** Every pixel earns its place. Strict 4px grid, aligned numeric columns, tabular figures, hairline (1px) borders, consistent corner radii. No glow, no neon, no heavy gradients, no drop-shadow haloes, no glassmorphism.
- **Dark, but not black-hole dark.** Layered surfaces separated by small steps in lightness and hairline borders, not shadows.
- **Data is the hero.** The pitch, the timeline and the numbers carry the visual weight. Chrome recedes.
- **Progressive disclosure.** Lead with the headline (a brief, a rating, a badge), reveal formulas and provenance on demand. Never force the maths on someone who wants the answer, never hide it from someone who wants to check it.
- **Explainability is a first-class element, not a tooltip.** A rating always has its working one click away (for example `V(actual) ÷ V(best in ring) = 0.18 ÷ 0.42`).
- **Restrained motion.** Short, purposeful transitions (150-200 ms). Replay animates because replay *is* motion. Honour `prefers-reduced-motion`.
- **Desktop-first.** Design at 1440 px wide; must hold at 1280 px. No mobile layouts in v1.

### Brand constraints (fixed)

| Item | Rule |
|---|---|
| Brand accent | `#1E76FA` (Ponta blue). Used for the logo, focus rings, selection, links' underlines, and the single primary interactive colour. |
| Typeface | **Geist** for UI text, **Geist Mono** for labels, IDs, timestamps and numeric data. Self-host; no Google Fonts CDN. Fallbacks: Inter, JetBrains Mono. Use `font-variant-numeric: tabular-nums` on all numeric columns. |
| Logo and mark | Use the supplied assets as-is. Do not redraw, recolour or re-type them. On the dark platform use the **white logo** (`Ponta logo_White.png`) and the **blue icon** (`Ponta icon_blue.png`). Files are in `ponta-blueprint/assets/` and in the `Ponta-Website` repo's `assets/`. |
| Hexagons | Only where they carry meaning: the pitch's hexagonal analysis grid (pressure, control and value overlays are computed per hexagon). No hex-shaped buttons, hex avatars or hex wallpaper. |
| Contrast | Body text must meet WCAG AA (4.5:1). `#1E76FA` is 4.17:1 on near-black: fine for graphics and large text, **not for body-size text**. Use `#4A90FB` for blue text on dark. White text on a `#1E76FA` fill is only 4.18:1, so if you use a filled primary button, choose a fill that passes (for example `#1558C9`) or use an outlined or tinted button. Verify, don't assume. |

### Colour: proposed starting tokens (the platform's own dark palette)

These are **proposals** for you to refine; expose them as CSS custom properties so they can be tuned centrally. The prior prototype used `#1a1a2e` (navy-tinted). The website's dark chrome uses neutral `#0A0A0A`. Choose a neutral or lightly cool near-black and justify it.

| Role | Starting value | Notes |
|---|---|---|
| Canvas (app background) | `#0B0D12` | |
| Surface 1 (panels, cards) | `#11141B` | |
| Surface 2 (raised, hover, menus) | `#171B24` | |
| Hairline border | `rgba(255,255,255,0.08)` | Stronger on interactive elements: `0.14` |
| Text primary | `#EDEDED` | |
| Text secondary | `#C3C3C3` | |
| Text tertiary | `#A7A7A7` | |
| Text quaternary (minimum) | `#8A8A8A` | Do not go dimmer for any text |
| Accent (graphics, focus) | `#1E76FA` | |
| Accent (text on dark) | `#4A90FB` | |

**Analytics colour semantics (these must mean the same thing on every screen):**

*(v1 values below. The prototype built from this brief changed them: home is `#d6293d` and away is teal. Reconcile in brief v2; the final away colour is still open.)*

| Meaning | Starting value |
|---|---|
| Home team | `#E63946` |
| Away team | `#1D8CF8` |
| Event highlight | `#FFD700` |
| Pitch | `#2D6A2D` (adapt for the dark UI; the pitch is a dark, desaturated surface, not a bright green slab) |
| Heat (pressure, value) | A perceptually uniform, colour-blind-safe gradient. You choose and justify it. |

**Three colour collisions you must resolve and explain (this is part of the task):**

1. **Away-team blue vs brand blue.** `#1D8CF8` and `#1E76FA` are nearly identical. Brand blue marks interaction; away-team blue marks a team. On a pitch with player dots, they will be confused. Propose a solution (for example a different away colour, or confining the brand accent so it never appears on the pitch canvas) and apply it consistently.
2. **Home-team red vs error and Blocked states.** Red already reads as "error". Do not use red for Blocked or for errors in a way that can be mistaken for the home team.
3. **Decision-quality bands vs parity states.** Decision-quality bands (Poor, Suboptimal, Good, Optimal) have traditionally used traffic-light colours (red, amber, lime, green). Parity states (Full, Degraded, Blocked) must **not** reuse the same traffic-light language, or the two will be read as one scale. Give parity states their own visual system (shape, fill pattern, icon, plus a restrained hue), and keep traffic-light colours for decision quality only, labelled in text as well as colour.

### Three required visual channels (do not merge them)

1. **Heat** (pressure, value overlays on the pitch): its own gradient.
2. **Confidence** (how reliable a frame, event or number is): its own channel, which must not reuse the heat colours. Suggested: texture, hatching, stipple or opacity rather than hue.
3. **Parity or tier** (Full, Degraded, Blocked, plus the match's data tier): a badge system with icon and shape, never colour alone.

Event-only decision-quality scores must look **structurally different** (for example dashed outline or hatched fill and a distinct label), not the same number with a smaller badge.

## 4. Pages and content

Build these screens, in this priority order. Navigation: persistent left rail (or compact top bar plus left rail) with the org switcher, primary nav, and user menu. All screens share one shell.

### 4.0 App shell and sign-in

- **Sign-in screen:** email and password and "Continue with SSO" (visual only). Ponta logo. No marketing copy beyond a one-line product description.
- **Shell:** org switcher (clubs can have several teams or squads), nav (Matches, Ask Ponta, Settings; greyed "Players", "Shortlists", "Team dashboards" marked *Coming later*), global search (visual only), user menu, a small environment badge ("Prototype, sample data").
- **Empty states** for a new org with no matches (clear next step: upload a file).

### 4.1 Match Library (the home screen)

Browse and manage matches.

- Table (default) and compact card view toggle. Columns: fixture, competition, date, **provider(s)**, **data tier badge**, **parity summary** (for example "9 Full, 2 Degraded, 3 Blocked"), processing status, brief status, last opened.
- Filters: competition, team, date range, provider, tier, parity state, status. Saved filter chips.
- Per-row quick actions: open brief, open replay, view parity report, re-process, delete.
- A prominent **Add data** action leading to the upload flow (4.2).
- Processing states: queued, processing (with stage names: parsing, normalising, quality check, analysing, writing the brief), ready, needs attention, failed (with plain-language cause).
- Mock content: 8-12 matches across several providers, including one event-only match, one GPS-only session, one multi-source (Tier 2/3, shown as locked or *In development*).

### 4.2 Add data (upload and connect)

A multi-step flow, designed to feel safe and clear:

1. **Choose source:** drag-and-drop upload (accepts the vendor's native exports) or "Connect an account" (visual only; shown as *Coming soon* for providers not yet supported). Show provider categories (optical tracking, GPS / wearables, event data), not logo walls. Name specific providers only in a "Supported formats" list: Metrica, SkillCorner, StatsBomb (events), STATSports (GPS).
2. **Ingestion preview and quality check:** what was detected (provider, format, sampling rate, duration, teams, player count, dropped frames), with any mismatches flagged (for example "these files do not look like the same match").
3. **Parity onboarding gate (4.3).**
4. **Confirm and process,** with a progress view.

### 4.3 Parity onboarding gate (a hero screen)

Shown after upload and reachable any time from a match. It explains in plain language what this data can and cannot support.

- A match header with provider, tier and processing summary.
- A **metric-family grid**: pressure, control zones, pass decision quality, carry decision quality, off-ball movement value, physical load, fatigue-related context. Each shows state (Full, Degraded, Blocked), a one-sentence reason with the measured figure when relevant ("Sampled at 10 Hz, 4.2% frames occluded"), and what would change it.
- A "What a second source would unlock" panel that is inviting, not apologetic: it shows the path from Tier 1 to Tier 2 to Tier 3 and is the natural prompt to add data. Tier 2/3 capabilities are shown as *In development* where not available, never as live features.
- A primary action: "Continue to brief".

### 4.4 Match AI Brief (a hero screen, and the shareable artefact)

A one-page brief for coaching staff, designed to read well on screen **and** export well as a PDF.

- **Header:** fixture, date, **tier and parity badge**, and a line stating which metrics are Degraded or Blocked for this match. For event-only or low-confidence data, this is stated at the top.
- **Body:** 5-8 short bullet insights grouped by theme (build-up, pressure, transitions, individual decisions). Every bullet is followed by one or more **citation chips**. Nothing is uncited. A bullet that cannot be cited does not appear.
- **Citation chip anatomy:** compact, mono-spaced, for example `evt_99482 · 34:12 · conf 0.91`. Clicking opens **Pitch Replay at that exact moment**. Hover shows a small preview of the moment. Low-confidence citations are visibly distinct (use the confidence channel).
- **"AI-generated" marking:** the whole brief is visibly marked as AI output with the model version and generation time. A short "How to read this" disclosure explains citations and confidence.
- **Actions:** export PDF, share link (visual only), regenerate, "Ask a follow-up" (opens Ask Ponta with context).
- **Print or export view:** a clean variant of the same layout on a light page, for printing. This is the only light-themed surface in the product.

### 4.5 Pitch Replay (a hero screen)

The product's visual signature. A 2D pitch canvas showing both teams' players and the ball in motion.

- **Pitch canvas:** accurate proportions (105 × 68 m default), dark desaturated surface, restrained line work. Players as team-coloured dots with jersey numbers. Ball. Event "ghost" markers.
- **Overlay toggles:** control zones (who can reach each zone first), value surface (heat), pass corridors, carry alternatives. Overlays drawn on a **hexagonal grid**. A visible legend that explains each overlay in one line.
- **Timeline scrubber:** play, pause, speed, frame step, event ticks on the timeline (passes, carries, shots), and **jump to citation** markers coming from the brief.
- **Event list (right panel):** the selected event with its **explainability card**: decision-quality rating with band label, the working (for example `V(actual) ÷ V(best in ring) = 0.18 ÷ 0.42`), the ranked alternatives, and the confidence. Event-only scores use the distinct visual class.
- **Player table:** live per-player values for the current frame (speed, distance, pressure received, control share), in a dense, aligned, mono-figure table.
- **Confidence overlay:** a toggle showing per-frame confidence on its own visual channel.
- Prototype note: animate mock trajectories (22 players and ball) smoothly. A canvas or SVG implementation is fine; performance at 25 Hz playback matters.

### 4.6 Ask Ponta (a hero screen)

A natural-language explorer.

- A large input with example prompts as chips ("Where did we lose the ball under high pressure in the second half?").
- Response layout: (1) **parity statement first** when relevant ("Wearable data isn't connected for this match, so this uses optical pressure only"), (2) a short summary, (3) a **playlist of replay chips** (each a citation chip with a thumbnail of the moment), (4) **named thresholds** shown inline ("high pressure = pressure index ≥ 0.7"), each clickable for its definition.
- **Hard rule:** the interface never shows a free-form number computed by the AI. Every number comes from a cited event and is shown inside a chip or a table row.
- A refusal state: when a question needs Blocked data (for example fatigue or injury questions), the answer explains why it cannot be answered and what is available instead. No speculation.
- A thread history sidebar. Streaming-answer animation (text arrives progressively; chips resolve after verification, with a brief "verifying citations" state).

### 4.7 Settings (lightweight)

Organisation profile, members and roles (visual only), data connections list, data retention and privacy summary (placeholder text), billing placeholder. Keep this restrained; it should look finished but is not a design priority.

### 4.8 Stretch screen (only if the above is complete): Player Profile

Identity card, on-pitch intervals, physical profile, a headline decision-quality number with band, and a combined timeline of decision quality and off-ball movement value. This is the screen most likely to be screenshotted into a report, so it should look credible and polished. Mark it clearly as *Later* in the navigation.

## 5. Content and claim rules (non-negotiable)

These are legal and product-integrity rules, not style preferences.

- **Mock data only, and obviously labelled.** Use a "Sample data" indicator. Base fixtures on Metrica Sample Game 3 style data (open sample data). **Player names must be fictional or pseudonymous** (for example "Home 7"); never use real players' names.
- **No medical, injury or causal-fatigue claims.** Physical or physiological context is shown as *association only*, with confidence bands and wording such as "coincided with" or "is associated with", never "caused by" or "will lead to". No injury prediction. No live or real-time claims; the product is post-match.
- **No invented proof:** no fake customer logos, testimonials, awards, press quotes or pricing. No claims such as "GDPR compliant" or "certified".
- **Tier 2 and Tier 3 are in development.** Where they appear, label them *In development* or *Early access*, with locked or preview states. Do not present fused or fitness-linked outputs as live.
- **Unsupported providers:** only Metrica, SkillCorner, StatsBomb and STATSports are real. Others (Catapult, Second Spectrum, Hawk-Eye, TRACAB and so on) may appear only as *Coming soon*.
- **AI output is always visibly AI.** Cited events one click away, limitations stated, never presented as a ground-truth rating.
- **British English** in all copy.
- **Tone of copy:** plain, specific, confident. No hype words ("revolutionary", "game-changing"), no exclamation marks, no emoji, no em dashes.

## 6. Components to design (a small, consistent system)

Define as reusable components with all states (default, hover, focus, active, disabled, loading, error):

Buttons (primary, secondary, ghost, destructive), inputs, selects, tabs, segmented controls, tables (dense, sortable, sticky header, tabular figures), filter chips, **tier badge**, **parity badge (Full, Degraded, Blocked)**, **confidence indicator**, **citation chip**, **AI-output marker**, **decision-quality band label**, toasts, modals and drawers, tooltips, empty states, skeleton loaders, progress and stage indicators, timeline scrubber, legend.

Build on an accessible primitive library (Radix primitives, shadcn-style) so keyboard and screen-reader basics come for free. Visible focus rings (brand blue) on every interactive element.

## 7. Technical requirements for the prototype

- **React + TypeScript**, structured so it can later live in a Next.js app. **Must work as a static export:** no server components fetching data, no server-only APIs, no secrets. Routing client-side with a single-page fallback for deep links (for example `/matches/123/replay`).
- Styling: Tailwind CSS (or equivalent) driven by CSS custom properties for the tokens above. No hard-coded colours in components.
- All data from local JSON mock files behind a thin data-access layer, shaped like an API (so a real API can replace it later).
- UI strings go through a simple i18n wrapper (one locale, English) from the start.
- Fonts self-hosted (Geist, Geist Mono).
- Runs with a single command (`npm install && npm run dev`) and builds with `npm run build`.
- Desktop only (1280-1920 px). No mobile layouts.
- Accessibility baseline: semantic landmarks, keyboard navigation for all primary flows, AA contrast verified on text, never colour-only meaning.

## 8. Out of scope for v1

Mobile layouts, a full design-system documentation site, real authentication, real data processing, real AI, billing, light theme (apart from the printable brief), localisation beyond English, video, and any screen not listed in section 4.

## 9. Report back

When done, provide:

1. The running prototype and the build instructions.
2. **Screenshots** of every screen in section 4 at 1440 px, and the key states (processing, event-only match, Blocked metric, low-confidence citation, refusal in Ask Ponta).
3. The final **token table** (colours, type scale, spacing, radii) and the contrast ratios you verified.
4. How you resolved the **three colour collisions** and the **three visual channels**, with reasoning.
5. A list of **assumptions you made** and **anything in this brief that was ambiguous, contradictory or hard to implement**.
6. Three alternative visual directions you considered and rejected, in one sentence each.

---

## Review checklist for v2 (founder use, not part of the prompt)

When reviewing the first prototype, judge it against these points, then fold the answers back into this brief as v2:

- **Does it look different from the website?** The platform should feel like a separate, dark, denser product with the same brand DNA.
- **Is it precise?** Check alignment, numeric columns, spacing consistency, type scale discipline, border and radius consistency.
- **Do parity states read as their own system**, separate from decision-quality bands and from home/away colours?
- **Is the citation chip right?** Is it compact and clear, and does it clearly lead to a moment?
- **Is the honesty working?** Does a Blocked metric feel like a trustworthy statement rather than a failure?
- **Which of the three alternative directions** the agent rejected is worth a second look?
- **What did the agent assume** that should become an explicit rule in v2?
- **Pitch Replay realism:** is the pitch the visual signature, or does it look like a generic chart?
- **Anything that overclaims**, makes medical or causal statements, or invents proof. Remove it and tighten the claim rules.
- **Decisions still open:** final dark palette (neutral vs lightly cool), away-team colour, heat gradient, whether primary buttons are filled or outlined, and Plotly vs Canvas for the pitch.

## Source documents (for the founder, and for the agent if it has repo access)

- `frontend.md`: information architecture and the hero views.
- `visual-design.md`: the website's visual language, colour tokens and contrast findings.
- `data-tiers.md`: data tiers and the Parity Report.
- `ai-layer.md`: citation rules, the semantic event schema and AI guardrails.
- `engine-governance.md`: claim policy for physiological and load-related output.
- `go-to-market.md` and `features.md`: market segments and personas.
- Logo assets: `ponta-blueprint/assets/` (and `Ponta-Website/assets/`).
