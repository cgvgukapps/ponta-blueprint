# Ponta Platform: Frontend Design Brief (v2)

**Version:** v2, written 15 November 2026 after the review of the first prototype. It supersedes [frontend-design-brief-v1.md](frontend-design-brief-v1.md), which is kept as history.

**How to use this file.** Everything from "1. Context" down to "8. Final report" is the prompt for the agent doing the next round. The last two sections are for the founder only. Where this brief and a source document disagree, this brief wins for the prototype, and the disagreement should be reported.

**What changed since v1.** v1 asked for a prototype from nothing. One now exists and has been reviewed and corrected. v2 therefore does two things: it **fixes the decisions** that v1 left open (section 3), so they are not reopened, and it **scopes the next round of work** on the existing prototype (section 5). The backend is still not connected; that is a later, separate task.

---

## 1. Context

**Ponta** is an AI-native football intelligence product for clubs, academies, federations and consultancies. A club uploads data from the providers it already uses (optical tracking, GPS or wearable data, event feeds). Ponta then does three things:

1. **Matrix**, the data engine, normalises what it is given into one quality-scored dataset and checks what that data can support.
2. **Plaiq**, the decision engine, turns it into football events (passes, carries, off-ball runs, defensive actions) and scores the **quality of each decision** against the options the player had.
3. **Grounded AI** reads those events to write a match brief and answer questions. Every statement links to the moment it came from.

Inside the app this is one product, "Ponta". The names Matrix and Plaiq appear only in technical detail panels.

**The idea that drives the interface is honesty about data.** For every match Ponta produces a **Data check**: for each metric family, the state is **Full**, **Degraded** or **Blocked**, with a plain-language reason and the measured figure. The brief and Ask Ponta only use what the Data check allows. A Blocked metric is stated plainly, never estimated. The interface should make this feel like a strength.

**What a club can bring** sets how much Ponta can say:

| Sources (UI name) | What the club provides | Status |
|---|---|---|
| **One source** | One provider's export for a match | Live in the product design |
| **Two sources** | Two sources for the same player group that are not time-aligned (for example old and new GPS during a vendor switch) | *In development* |
| **Paired sources** | Two or more sources for the same match, aligned to one clock (for example GPS plus optical) | *In development* |

Internal documents call these Tier 1, 2 and 3. **The interface never says "Tier" or "parity".**

**Users, in priority order:** the performance or video analyst (primary: hours in the app, wants density and precision); the head coach and coaching staff (reads the brief in under two minutes, clicks citations); the sports scientist (reads physical context, only where a GPS source exists; the product makes no medical claims). Scouts and directors of football come later.

**Market.** Clubs below the top divisions in major markets, top divisions in smaller ones, and clubs on lower-cost tracking. Real data, no data-science team, sceptical of black-box ratings.

### The existing prototype (start here)

- Repo: `~/Ponta_Platform_UI` (GitHub `cgvgukapps/ponta-platform-ui`), `main` at `a2d3e62`. Tag `prototype-v0.1` marks the first build.
- Stack: Vite, React, React Router, TypeScript, Tailwind on CSS custom properties, Radix primitives. Static build with an SPA fallback.
- Read first: `README.md`, `docs/design-notes.md` (tokens, the colour decisions, assumptions, and section 6 for the review changes), `docs/contrast.md`, and the screenshots in `docs/screenshots/`.
- Checks that must keep passing: `npm run typecheck`, `node scripts/check-i18n.mjs`, `npm run contrast`, `npm run build`, `npm run shots`.

## 2. Objective

Take the reviewed prototype to the version that design partners can be shown. Keep everything in section 3 as it is. Deliver the changes in section 5: names rehydrated in the client, a coverage figure in the Data check, a standalone shareable match page, and the listed fixes. Everything stays mock data behind the existing API-shaped data layer.

## 3. Fixed decisions (do not reopen without reporting why)

These were open in v1 and are now decided, either by the first prototype or by the review. The values live in `src/styles/tokens.css`; change them there only.

*(Revised 16 November 2026: visual redesign to the founder's references, a calm neutral tool UI and simple, high-impact components. Rows marked "revised" supersede the first values, which are in git history. Recorded in the UI repo's `docs/design-notes.md`, section 7.)*

### Palette and type

| Decision | Value |
|---|---|
| Dark palette (revised) | Neutral near-black. Canvas `#161616`; surfaces `#1c1c1c`, `#242424`, `#2a2a2a`. Panels are fills with no outline; hairlines (white 7%) only divide lists |
| Text (revised) | `#f5f5f5`, `#cfcfcf`, `#a3a3a3`, `#979797` (the floor; nothing dimmer) |
| Brand accent (revised) | `#1e76fa` for state only: focus, selection, toggles, progress. Never a button fill. Blue text on dark is `#5c9dfc` |
| Primary button (revised) | **White fill** `#f5f5f5`, near-black label `#111111`. Secondary: soft fill, no outline |
| Home team | Crimson `#d6293d`, circles and team dots only |
| Away team | **Teal `#2bc7b9`**. Brand blue never appears on the pitch |
| Event highlight | `#ffd700` |
| Pitch | `#16231b`, lines white 26%, hex grid white 7% |
| Heat | Lightness-monotonic purple to pink ramp, `--heat-0` to `--heat-8` |
| Type (revised) | **Geist only**, self-hosted, body 14 px, page titles 26 px, key figures 36 to 40 px; tabular figures on every number. Geist Mono only for code. No uppercase labels: section labels are sentence case |
| Radii (revised) | 6, 8, 12 and 18 px |
| Pitch rendering | **HTML canvas**, not Plotly |
| Icons (new) | Thin line icons, 1.5 px stroke |
| Key figures (new) | One large figure with a small grey label beneath, in a soft tile (decision quality in the replay, Data check counts) |
| The one flourish (new) | The Ask composer's gradient edge, blue `#1e76fa` to rose-red `#f0436b`. Used nowhere else; rose-red rather than red so it is never read as the home team or an error |

### The three channels and the colour collisions

- **Heat** carries magnitude (value, pressure) on the pitch only.
- **Confidence** carries how certain the tracking is: a level bar, a dashed edge, and fading. No hue.
- **Data check states** use one stone hue and read as how full the mark is: solid square for Full, light fill with outline for Degraded, empty outline for Blocked.
- **Decision-quality bands** are the only place traffic-light colours appear, and each band also has a bar glyph and a text label.
- Home red is a circle; errors are rectangular banners with an icon; Blocked is never red.

### Decision quality: the scale and the bands (from the engine, not the prototype)

- The score is **the value where the ball arrived divided by the best option available when it was played**. It runs from **0 to 1.5**, not 0 to 1. A score above 1.0 means a run made the outcome better than any option at release.
- Bands, unified across passes, carries and intercepted passes: **Poor below 0.55, Suboptimal 0.55 to 0.79, Good 0.80 to 0.94, Optimal 0.95 and above**. Source: `PONTA-PROTOTYPE/ANALYTICS_SPECIFICATION.md` section 7.1. Thresholds come from `src/lib/bands.ts`, never hard-coded.
- Wherever a score is shown large, show it on the 0 to 1.5 scale with the cut-offs marked. Never clip at 1.0 and never present the score as a percentage.
- The working is shown as `V(at arrival) ÷ V(best at release) = a ÷ b = score`.
- **Event-only** scores are a range with a dashed outline and ruled fill (`Estimate · Poor to Suboptimal`), never a point.
- Goalkeepers and set pieces are shown as **Not scored**, with the reason.

### Citation chip anatomy

`▶ 34:12 · Pass, Home 10 to Home 8 · ▬ 0.91`

- It leads with what a coach reads: the minute, then the action.
- The confidence comes last, as a level bar plus a number. Low confidence adds a dashed outline (the confidence channel), never a hue.
- The **event id** is kept for audit: in the hover preview, in the accessible name and on the printed brief (`[1] 07:48 · Pass, Home 5 to Home 6 · 0.94 · evt_41207`).
- Use the **compact** chip (minute and confidence only) where a caption already names the action.
- Clicking a chip opens Pitch Replay at that moment.

### Naming in the interface

| Use | Not |
|---|---|
| Data check | Parity, parity report, metric parity |
| One source, Two sources, Paired sources | Tier 1, Tier 2, Tier 3, data tier |
| Full, Degraded, Blocked | Traffic-light words for these states |
| Decision quality, with its band | Rating, IQ score, accuracy |
| In development | Coming soon (for capabilities); Coming soon is only for providers |

## 4. Constraints

### Content and claim rules (non-negotiable)

- **Mock data only, visibly labelled.** Keep the "Prototype, sample data" badge.
- **No real player names anywhere.** Rehydrated names in this round (5.1) must be fictional and generated, and must not resemble real players.
- **No medical, injury or causal-fatigue claims.** Physical context is association only ("coincided with", never "caused by"). No injury prediction. No real-time or live claims; Ponta is post-match.
- **Providers:** only Metrica, SkillCorner, StatsBomb and STATSports are supported. Others (Catapult, Second Spectrum, Hawk-Eye, TRACAB and so on) appear only as *Coming soon*.
- **Two sources and Paired sources are *In development*.** Show what they would unlock; never show fused or fitness-linked output as live.
- **No accuracy claims for decision quality.** It is a defined, explainable score. Do not write "accurate", "validated" or "proven" about it.
- **AI output is always marked as AI**, with the model version and time. The interface never shows a number the model computed; every number comes from a cited event or a table row.
- **No invented proof:** no customer logos, testimonials, pricing, certifications or "GDPR compliant".
- **Copy:** British English; plain, specific and confident; no hype words, exclamation marks, emoji or em dashes.

### Technical

- Keep the stack and the static build. Do not move to Next.js (the framework decision is open, leaning Vite).
- **Do not connect a backend.** Keep every `src/api` function signature; new data goes behind new functions of the same async, JSON-shaped kind.
- No hard-coded colours in components; tokens only. Every new string goes through `src/i18n/en.ts`.
- Keep the fake engine (`src/lib/engine.ts`) and the mock packs out of anything described as production. They are prototype-only.
- Desktop only, 1280 to 1920 px. Accessibility baseline as v1: landmarks, keyboard paths for primary flows, AA contrast verified, never colour-only meaning.

## 5. Deliverables

Work in this order. Each item lists what done looks like.

### 5.1 Names rehydrated in the client

The AI layer only ever sees pseudonyms (`home_5`); real names are added by the client from the club's own roster. The interface must be designed for that.

- Add a mock roster per match with **fictional** names: include long names (over 24 characters), names with diacritics, and at least two in non-Latin scripts.
- Brief text, Ask answers and captions carry **reference tokens** (for example `{home_5}`) that the client replaces with a name, or with the pseudonym when no roster entry exists.
- Add a workspace toggle, **Show names / Show codes**, in the user menu.
- Check every screen with names on: tables, chips, the explain card, the player table, the pitch tooltip and the print view. Truncate with a full-name tooltip; never let a name break a numeric column.
- Pitch markers keep shirt numbers, not names.
- **Done when:** every screenshot can be captured in both modes with no overflow, and no pseudonym or name is hard-coded in brief or answer text.

### 5.2 Coverage figure in the Data check

A metric can be Full while a share of events could not be scored (ball not detected, goalkeeper, set piece). The Data check must say so.

- Add a **Coverage** stat to the Data check header: the share of passes and carries scored, for example `81% of passes and carries scored`.
- Underneath, list the causes with counts: ball not detected, goalkeeper, set piece, event outside tracked frames.
- Show the coverage figure in the brief header too.
- Give each mock match plausible mock figures. The real engine will measure them; the prototype only designs the slot.
- **Done when:** the Data check, the brief and the print view all show coverage, and a match with low coverage reads as a stated limit, not an error.

### 5.3 Shareable match page (single file)

Ponta's first real deliveries will be founder-run: a prospect sends one export, and Ponta sends back the Data check and a cited brief as a page. Design that page.

- A standalone, read-only page per match: header, Data check summary with reasons, coverage, and the brief with citation chips. Light theme, built from the print view.
- It renders from **one JSON file** per match. Document its shape in `docs/match-page-payload.md`, reusing the field names of the semantic event schema in `ponta-blueprint/ai-layer.md` (`event_id`, `timestamp_s`, `dq_score`, `dq_band`, `player_ref` and so on) and the Data check fields already used by the prototype.
- Citation chips on this page cannot open a live replay. Each chip opens a static moment thumbnail with the explain card instead.
- It works when opened from disk or a static host, with no sign-in.
- **Done when:** `npm run build` also writes one such page for `m_0412` that opens offline, and the payload document exists.

### 5.4 Fixes carried from the review

1. **Error colour.** `--danger` and `--dq-poor` are the same orange-red, as are `--warn` and `--dq-suboptimal`. Move `--danger` and `--warn` to hues that cannot be read as a band (for example a magenta-red for danger), then re-run the contrast check.
2. **Print.** Verify pagination of the print brief in Chrome and Safari: no chip split across pages, the header repeated, and page numbers present.
3. **Pitch accessibility.** Keep the canvas as one image for screen readers, but add a keyboard path: arrow keys step through events, and the explain card announces the selection. The player table remains the accessible view of player values.
4. **Video slot.** Keep the disabled Video button next to "Play from 5 s before". Add a Settings placeholder, "Linked video source", marked *In development*. Do not design a video player.
5. **Data check reasons.** Every Degraded or Blocked row must carry a measured figure or a named cause. List any that do not.

### 5.5 A mapping for later

The backend is not being connected now, but the mock shapes should not drift further from the planned API. Write `docs/api-mapping.md`: for each `src/api` function, the planned endpoint and the fields it will return, mapped to the semantic event schema and the Parity Report. Where the mock uses different names, record the difference; do not refactor yet.

## 6. Acceptance criteria

- Everything in section 3 is unchanged, and the `npm run contrast` check passes after the colour change in 5.4.
- Deliverables 5.1 to 5.5 are met, with their "done when" conditions.
- `typecheck`, `check-i18n` (no missing keys), `contrast`, `build` and `shots` all pass. Screenshots are re-captured, including names on and off.
- No string in the interface contains "Tier" or "parity", except inside technical detail panels.
- No real player names, no provider shown as supported other than the four, and no medical or causal wording anywhere.

## 7. Explicitly not in scope

Connecting a backend or real authentication; generated API types; moving to Next.js; mobile layouts; a video player; player comparison, shortlists and team dashboards; billing; localisation beyond English; any change to the engine maths, band thresholds or the meaning of a Data check state.

## 8. Final report

Add a section 7 to `docs/design-notes.md` (the next free section) covering:

1. What changed, screen by screen, with before and after screenshots for 5.1 and 5.2.
2. The new colour values from 5.4 and their contrast ratios.
3. The payload document (5.3) and the API mapping (5.5), linked.
4. Every assumption made, and anything in this brief that was ambiguous, contradictory or hard to implement.
5. Any rule in section 3 or 4 you think is wrong, with the reason. Do not change it; report it.

---

## Founder review checklist for v3 (not part of the prompt)

- **Names on:** does the product still read as precise and calm with real-length names? Does any screen depend on short pseudonyms?
- **Coverage:** does a low-coverage match read as honest or as broken? Is one figure enough, or does each metric need its own?
- **Shareable page:** would you send this page to a prospect after a concierge run? What is missing for that conversation?
- **Error colour:** can Poor and an error still be confused anywhere?
- **Show to two analysts** before v3: can they read a decision-quality score above 1.0 without explanation? Do they understand "Data check" without the word parity?
- **Still open:** Vite or Next.js; whether event-only analysis ships in the first release; the public naming of Matrix and Plaiq; which video source to link first.

## Source documents

- `docs/design-notes.md` in the UI repo: tokens, decisions and the review changes.
- [frontend.md](frontend.md): information architecture and the hero views.
- [data-tiers.md](data-tiers.md): sources, the Parity Report (Data check in the interface) and coverage.
- [ai-layer.md](ai-layer.md): citation rules, the semantic event schema and pseudonymisation.
- [engine-governance.md](engine-governance.md): claim policy and construct validity of decision quality.
- `PONTA-PROTOTYPE/ANALYTICS_SPECIFICATION.md` section 7: the decision-quality formula and bands.
- [go-to-market.md](go-to-market.md) and [roadmap.md](roadmap.md): the concierge step behind 5.3.
