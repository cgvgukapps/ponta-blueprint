# Ponta Legal & Compliance

See [README.md](README.md) for baseline assumptions. **Nothing in this document is legal advice** — it's a realistic plan for *when and how* to get real legal advice, sized appropriately for a solo/bootstrapped startup, not a substitute for it.

## Why this is a sports-specific concern, not generic SaaS boilerplate

Most B2B SaaS legal checklists (ToS, privacy policy, a DPA template) apply here too, but PLAIQ has one risk category that's genuinely different from a typical SaaS product: it computes and potentially surfaces a **named, identifiable professional athlete's "Decision Quality" and "Player IQ" rating**. Player unions (FIFPro in particular) have actively contested who owns and can commercialize player performance data, and a rating that's leaked, misquoted, or shown to be wrong on a specific high-profile decision carries reputational and legal exposure that a generic analytics dashboard doesn't. This deserves specific attention, not a checkbox.

## What a realistic startup actually does, and when

**Before any paying pilot (do this early, it's cheap):**

- A fixed-scope initial consultation (a few hours, not a retainer) with a lawyer who has sports-data or data-privacy experience, covering three specific questions: (1) GDPR/UK-GDPR posture for processing performance data tied to identifiable individuals across the EU/UK leagues most likely to be early customers, (2) whether publishing a performance rating about a named player creates defamation-adjacent exposure and what disclaimers/framing mitigate it, (3) a sanity check on the vendor Terms of Service question below.
- **Read each data vendor's ToS for redistribution/derived-works restrictions.** StatsBomb, SkillCorner, and the others license data at the league/season level (`architecture.md`'s licensing-scope-awareness note already reflects this), and their terms likely constrain what a *third party* (PLAIQ) is allowed to do when accessing a customer's data via their API on the customer's behalf. This needs an actual read-through, not an assumption — get the lawyer's few hours of review to cover this alongside the GDPR question.
- **Adopt a standard SaaS ToS/DPA template, then get it reviewed, not drafted from scratch.** Off-the-shelf template services exist precisely for this stage of company — the cost-effective path is customize a template to state clearly that PLAIQ acts as a **data processor** on the customer's instruction (not an independent controller creating a new derived product), and that the customer warrants they have the rights to the data they connect or upload. This is a few hundred dollars and a couple of lawyer-reviewed hours, not a bespoke legal-drafting project.

**Before the first contract with a larger customer (revisit, don't rebuild):**

- Larger clubs' procurement/security teams will very likely run a vendor security questionnaire (this was already flagged in `architecture.md`'s credential-custody section) — have the DPA, a plain-language security summary (referencing the Supabase Vault / RLS / envelope-encryption design already in `architecture.md`), and answers to common questions (data retention, sub-processors, breach notification) ready *before* that conversation happens, not drafted under deadline pressure during it.
- Consider cyber liability / errors & omissions insurance at this point — not needed pre-revenue, but a reasonable trigger is "first contract with an organization large enough to have its own legal/security review process."

**Ongoing:**

- Treat FIFPro's and similar bodies' public positions on player data rights as something to monitor, not litigate against pre-emptively — the field is evolving, and overinvesting in speculative compliance before there's a real regulatory requirement is its own kind of waste for a solo founder's limited time.

## Custom-tool engagements need their own contract, not an extension of the standard ToS

`go-to-market.md`'s design-partner track includes building bespoke tools for a small number of early clubs, with the explicit intent of generalizing what's learned into the core product (`architecture.md`'s per-tenant module mechanism). The standard SaaS ToS/DPA above governs *data* rights (PLAIQ as a processor acting on the customer's instruction) — it says nothing about who owns bespoke *software* built during an engagement, and that's a genuinely different question a standard SaaS template doesn't answer.

**What's needed before the first custom-tool engagement, not after:** a short services/statement-of-work addendum, covered in the same fixed-scope lawyer consultation as everything else above, that explicitly states:

- **PLAIQ retains ownership of the software, methodology, and any general tooling built during the engagement**; the club receives a license to use it (exclusive or not is a negotiable commercial term, but it must be a deliberate choice, not a silent default).
- **PLAIQ may incorporate general learnings and reusable tooling into the broader platform** — but never the club's specific confidential data, tactical preferences, or derived insights, without their consent. This is the same non-negotiable principle already stated in `architecture.md`'s licensing-scope-awareness note (never share one tenant's data with another), extended explicitly to cover derived custom-tool IP, not just raw data.

**Why this can't wait:** without an explicit clause, a club could reasonably assume — or explicitly demand — that a tool built specifically for them stays exclusive, which would block the entire point of the custom-tools track (learning from one club to improve the shared product). Cheap to settle in the first contract; expensive and relationship-damaging to renegotiate after a club believes something was built exclusively for them.

## Player identity pseudonymization

`architecture.md`'s credential & identity custody section (§6) specifies an `identity_registry` pattern: analytics tables, ML training data, caches, and logs reference an opaque `person_id`, never a name column directly, with plaintext name/DOB held in one column-encrypted table decrypted only at the API display edge. Worth stating here because it directly strengthens the FIFPro/named-athlete risk flagged above — pseudonymization is explicitly named in GDPR Art. 4(5)/32 as a recommended technical safeguard, so this is a legal-compliance benefit, not just a security-hardening choice.

This doesn't change the sequencing or legal spend in the table below — it's a schema-design decision to lock in at the first real Postgres migration, not a new compliance milestone requiring its own legal review. It's also not a substitute for the defamation-adjacent question in the fixed-scope consultation above: whether showing a Decision Quality rating for a named player creates exposure is a risk that exists whenever the rating is *displayed*, regardless of how the underlying name is stored — the two are complementary, not substitutes.

## Data retention & termination

Directly tied to the credential-custody design in `architecture.md`: define one clear policy (a common SaaS pattern — e.g., customer data retained for a fixed grace period after cancellation, then permanently deleted on request or automatically) and state it in the ToS/DPA rather than leaving it implicit. The engineering side of this (a purge job, credential revocation on disconnect) is scoped in [operations.md](operations.md#data-lifecycle--termination) — this document owns the *policy*, operations.md owns the *implementation*.

## Sequencing summary

| Milestone | What must exist |
|---|---|
| First design-partner/free pilot | Basic ToS covering acceptable use and liability limitation — low risk, but don't skip entirely |
| First custom-tool engagement | Services/SOW addendum with the IP-retention and generalization clause above — settled before starting, not after |
| First paid contract | Lawyer-reviewed DPA, vendor ToS review complete, retention policy stated |
| First contract with a larger/enterprise-scale customer | Security questionnaire response pack, insurance consideration revisited |

This keeps legal spend proportional to actual risk exposure at each stage, rather than front-loading a full compliance program before there's any revenue to justify it.

## Website contact form (built; goes live at launch)

[pontahq.com](https://pontahq.com) takes enquiries through Formspree (`https://formspree.io/f/xgaopwep`). This document previously had nothing on website forms; that gap is now a live fact, not a future one.

- **Public promise on the page:** "We'll use your details only to reply to you." That has to stay true. If outreach lists or a CRM enter the picture, the line has to change first.
- **Sub-processor:** Formspree processes submissions and retains them under its own policy. There is no Ponta privacy policy yet; the form links Formspree's.
- **Not a data-ingest path.** The form is name / email / optional providers / message. It does not accept file uploads. "We'll show you what Matrix returns" is a conversation, not an invitation to send player tracking files through Formspree. The FAQ questions "What do we need to send you?" and "What happens to our data?" are correctly **held** until a founder + lawyer answer exists.
- **Open-data naming.** The live proof and Plaiq worked example name Metrica and SkillCorner. Licence terms for that reuse are still unverified; accepted as non-blocking at launch, still needs a real check (see [website.md](website.md)).

## Vendor licence check before the first real export

*Added 15 November 2026.* The earlier plan reads vendor terms before the first paid contract. A concierge pilot (see [roadmap.md](roadmap.md#phase-2--tier-1-pipeline-and-ai-workflows)) receives a real export earlier than that, and a free pilot is still a third-party processing act. Before any prospect sends a real export:

- Read each supported vendor's customer terms for restrictions on giving licensed data, or data derived from it, to a third-party processor. Single-source use is not exempt: this applies to Tier 1.
- Check whether derived events sent to an LLM provider are allowed under the same terms, in addition to the sub-processor requirements below.
- Ask the prospect, in the first call, whether their licence permits it. Get the answer in writing with the export.
- Keep the Track A open-data position separate: licence terms for the Metrica and SkillCorner open samples are still unverified (gate G3 in [website-v2-plan.md](website-v2-plan.md)).

## AI and health-adjacent data

*Added with the AI-native revision. Nothing here is legal advice; these are the questions the fixed-scope lawyer consultation above must now also cover, and the triggers for each.*

### LLM provider as a sub-processor

The AI layer ([ai-layer.md](ai-layer.md)) sends selected, pseudonymised semantic events to an external LLM provider. That makes the provider a **sub-processor** that must appear in the DPA. Required before any customer data reaches a provider: a **no-training** commitment, **zero or tightly limited retention**, a data-processing agreement with the provider, and a stated region/transfer basis. Pseudonymisation (`home_2` instead of a name) is a real safeguard but **not anonymisation** — a known fixture, date, minute, role and opposition can allow re-identification — so the DPA and the product's privacy language must describe it as pseudonymisation, and prompt context is minimised to the selected events. Provider choice is abstracted so it can be changed or self-hosted if a customer's procurement requires it.

### Physiological and wearable data

Heart rate, accelerometer/gyroscope-derived load, and similar channels sit closer to GDPR special-category (health) data than positional coordinates do. Today the engine ingests none of these channels (the STATSports adapter ignores them), so this is **not yet a live exposure — it becomes one the moment Tier 3 or load-based Tier 2 ingestion is built.** Trigger: *before the first ingestion of heart-rate/IMU data from a real customer*, get explicit legal review of the lawful basis (including whether player consent is needed in addition to the club's), special-category conditions, retention, and the right of access/erasure for individual players. This also applies to the unlicensed STATSports sample used in the prototype: it must not be reused in the product (see `matrix-statsports-adapter-task.md`).

### Medical and injury claims

The product must not diagnose, predict injury, or give medical advice. Output on load and decision quality is framed as an **association for staff review** (see `engine-governance.md`'s claim policy). Terms of service and in-product text should state that outputs are decision-support analytics, not medical advice, and that clinical decisions remain with qualified staff. A fatigue "alert" worded as injury risk would change the regulatory and liability profile of the product; avoid it unless and until reviewed.

### Fusing vendors' data — now an earlier gate

`Matrix.md` flagged that no vendor's ToS has been checked for whether combining two vendors' data into a derived dataset is permitted even for the licensing customer's internal use. Under the previous plan this was a pre-Step-B question; because Tier 2/3 are now committed product directions, it moves up: **review before any Tier 2/3 feature is built against real data**, not before it ships. Tier 1 (single-source normalisation) carries none of this exposure.

### Named-athlete ratings in AI output

The defamation-adjacent and FIFPro-type exposure for named-player ratings (above) is unchanged, but AI-generated prose adds a new way to produce a damaging sentence. The citation and no-causal-claims rules in `ai-layer.md` are the mitigation; include AI-generated briefs in the lawyer's review of disclaimers and framing.

### Sequencing additions

| Milestone | Added requirement |
|---|---|
| Before any customer data reaches an LLM | LLM sub-processor terms (no-training, retention, DPA listing); privacy language describing pseudonymisation accurately |
| Before first heart-rate/IMU ingestion from a real customer | Health-data legal review; lawful basis and player-rights process |
| Before building Tier 2/3 against real data | Vendor ToS review for derived/fused datasets |
| Before shipping any fatigue/load-linked AI output | Medical-claims language and terms reviewed |
| Before the first real export from a prospect (including a free concierge pilot) | Vendor licence check on third-party processing and on sending derived events to an LLM provider; written confirmation from the prospect |
