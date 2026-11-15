# Ponta Operations

See [README.md](README.md) for baseline assumptions and [architecture.md](architecture.md) for the system this document operates. Covers what's needed to run the platform responsibly at solo/small scale — proportional in the same spirit as the rest of the blueprint: the minimum real version now, not the sophisticated version a funded team would build.

## API governance

`architecture.md` commits to "every engine capability maps to a documented endpoint" as a design principle, which means the API needs baseline governance even for v1, before any external (non-frontend) consumer exists:

- **Version the API from day one**: prefix every route `/v1/...`. Costs nothing now, avoids a painful retrofit the first time a breaking change is needed after a customer (or the frontend itself) depends on the contract.
- **Simple per-org API keys**, minted via Supabase, passed as a bearer token — not a full OAuth2 client-credentials flow. The latter is the "correct" enterprise answer but is disproportionate infrastructure before there's an actual external API consumer beyond your own frontend.
- **Basic rate limiting at the edge** (a library like `slowapi` for FastAPI, or rate limiting at the Fly.io/Cloudflare layer) — enough to prevent accidental abuse (a runaway frontend retry loop, a misconfigured script) without building a custom quota/billing-metering system prematurely.

Revisit all three with more rigor only when a real trigger exists: a paying customer building against the API directly, or the platform-licensed/API-product angle in `architecture.md`'s Future Evolution section becoming real.

## Data lifecycle & termination

Policy owned by [legal-compliance.md](legal-compliance.md#data-retention--termination); this section owns the implementation:

- A single **tenant purge job**: soft-delete on cancellation (data inaccessible but recoverable), hard-delete after a defined grace period (or immediately on explicit customer request) — covering Postgres rows, object storage files, and critically, the Supabase Vault credential entries from `architecture.md`'s credential-custody design (a revoked customer's vendor credentials should not silently persist in the vault after offboarding).
- Surface this as a real action in the product (Settings → org management, per `features.md`'s org/workspace management feature), not just a backend script run manually on request — a paying customer will eventually ask "can I export/delete my data," and that should have a real answer.

## Operational resilience for a solo operator

The honest answer for a company of one isn't "build enterprise HA infrastructure" — it's "be deliberate about the few things that actually matter at this scale":

- **Automated backups plus an actually-tested restore.** Supabase includes automated Postgres backups by default — the missing piece most solo operators skip is periodically *proving* a restore works, not just assuming the backup exists. Put a calendar reminder on this; it's a process gap, not a technical one.
- **Idempotent, retryable jobs.** RQ/Celery both support retries natively — design `process_match` so a transient failure (a flaky vendor API call, a momentary network blip) self-heals on retry rather than requiring manual intervention every time.
- **Uptime monitoring with actual paging**, via a free/cheap tier of a service like Better Uptime or UptimeRobot, so the founder finds out about an outage from a page, not from a customer's email.
- **Honest SLA language in contracts.** A solo-operator startup should not promise 99.99% uptime it can't realistically staff for; "best effort, target 99% uptime, business-hours support" is an honest commitment that won't be breached by the reality of being a team of one. Overpromising here creates contractual risk that a more conservative honest commitment avoids entirely.

## Unit economics

No document in this blueprint has put actual numbers behind the repeated claim that the architecture is "cheap at small scale" — that claim needs to survive contact with a real cost model before it's load-bearing for pricing decisions.

**What to actually build:** a simple spreadsheet (not more infrastructure) with two cost lines:

1. **Cost per match ingested** — object storage for raw + processed files (a full match's 25fps tracking data across ~22 players for ~90 minutes is a real but bounded size, on the order of tens of megabytes raw, more once xT surfaces/hex grids are cached), plus the one-time compute cost of the Tier-1 batch analytics job (`architecture.md` §4).
2. **Cost per active tenant per month** — baseline hosting (Fly.io/Render), Postgres/Redis at the relevant Supabase/Upstash plan tier, and marginal Tier-2 overlay compute from interactive usage (cached, so marginal cost per additional viewing session should be small per `architecture.md`'s Redis-caching design).

**What this is for:** setting subscription pricing with real gross-margin headroom (SaaS businesses typically target 80%+ gross margin) rather than a number picked by intuition. This is explicitly more urgent for the *platform-licensed* tier in `architecture.md`'s Future Evolution section, where data-licensing cost becomes a direct COGS line — BYOD tiers are lower-risk here since the customer carries their own data cost.

This document doesn't attempt to produce final numbers — current cloud/vendor pricing should be checked directly against provider pricing pages at the time this is actually built, since prices drift and a stale estimate here would be worse than no estimate.

## LLM costs and AI operations

*Added with the AI-native revision ([ai-layer.md](ai-layer.md)).* The AI layer adds a new cost line and new operational failure modes; the unit-economics spreadsheet above must include them before pricing is set.

- **Cost per brief.** One LLM synthesis call per match over a *bounded* set of deterministically selected events (not the whole match), generated once at ingestion and cached as an insight card. Cost scales with matches ingested, so it is a per-match COGS line alongside storage and batch compute.
- **Cost per explorer query.** Ask Ponta is interactive and unbounded by nature, so it needs hard limits: token caps per request, a tool-call cap per question, and per-org rate and monthly spend limits, enforced in the API (the existing `slowapi`/edge rate-limit guidance applies). A single heavy user must not be able to generate unbounded spend.
- **DuckDB-on-object-storage costs.** Query cost is dominated by object-storage reads and egress (R2's low egress is the reason it is preferred). Measure per-query bytes read on real matches; the Parquet layout (partitioning by match and period, column pruning) is the lever.
- **Pricing implication.** LLM cost makes per-seat flat pricing riskier than usage-aware tiers; decide whether Ask Ponta is capped, metered or bundled. This is a pricing decision for later, but the *measurement* must exist first.
- **Provider resilience.** Abstract the LLM behind one adapter; define a fallback model/provider and a degraded mode in which briefs fail soft ("brief pending") rather than blocking ingestion. A provider outage must never mark a match as failed — the deterministic store is complete without the AI layer.
- **Audit trail.** Store prompt version, model identifier, tool calls, citations and verifier result for every AI output, **without player names**, under the same retention/purge policy as other tenant data ([legal-compliance.md](legal-compliance.md#data-retention--termination)). This is also what lets a coach ask "why did it say that" and get a real answer.
- **Monitoring.** Alert on citation-verifier rejection rate, refusal rate, latency, and per-org spend, in addition to the existing uptime and job-failure monitoring. A rising rejection rate is the early signal of a prompt/model regression.
- **Purge job extension.** The tenant purge job must also delete both Parquet artifacts per match, insight cards, and AI audit logs, in addition to rows, files and Vault credentials.
