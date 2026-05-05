<!-- studiomeyer-mcp-stack-banner:start -->
> **Part of the [StudioMeyer MCP Stack](https://studiomeyer.io)**, Built in Mallorca · ⭐ if you use it
<!-- studiomeyer-mcp-stack-banner:end -->

<div align="center">

# n8n Templates · StudioMeyer Memory

**Drop-in n8n workflows that turn AI agents from amnesia patients into systems that remember.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![n8n compatible](https://img.shields.io/badge/n8n-2.10.1%2B-FF6E5C.svg)](https://n8n.io)
[![Custom Node](https://img.shields.io/npm/v/n8n-nodes-studiomeyer-memory?label=community%20node&color=blue)](https://www.npmjs.com/package/n8n-nodes-studiomeyer-memory)
[![Templates](https://img.shields.io/badge/templates-8%20live-brightgreen.svg)](#templates)
[![Memory backend](https://img.shields.io/badge/memory-studiomeyer.io-d4af37.svg)](https://memory.studiomeyer.io)
[![Memory-free variant](https://img.shields.io/badge/memory--free-n8n--workflows-555.svg)](https://github.com/studiomeyer-io/n8n-workflows)
[![CI](https://github.com/studiomeyer-io/n8n-templates/actions/workflows/validate-workflows.yml/badge.svg)](https://github.com/studiomeyer-io/n8n-templates/actions)

Voice agents · customer support · personal assistants · cross-session memory · multi-provider LLM

[Quick Start](#quick-start) · [Templates](#templates) · [Architecture](#architecture) · [Memory-free variant](#memory-free-variant) · [Ecosystem](./ECOSYSTEM.md) · [Contributing](./CONTRIBUTING.md)

</div>

---

## A note from us

We have been building tools and systems for ourselves for the past two years. The fact that this repo is small and has few stars is not because it is new. It is because we only just decided to share what we have built. It is not a fresh experiment, it is a long story with a recent commit.

We love building things and sharing them. We do not love social media tactics, growth hacks, or chasing stars and followers. So this repo is small. The code is real, it gets used, issues get answered. Judge for yourself.

If it helps you, sharing, testing, and feedback help us. If it could be better, an issue is more useful. If you build something with it, tell us at hello@studiomeyer.io. That genuinely makes our day.

From a small studio in Palma de Mallorca.

## Why this exists

Without persistent memory every AI interaction starts from zero. Voice agents forget callers. Support bots ask returning customers their email three times. Personal assistants miss the project context you discussed yesterday.

These templates fix that. Each workflow follows the same loop: **search the memory, reason with the context, write the outcome back**. Memory is provided by [StudioMeyer Memory](https://memory.studiomeyer.io), a hosted MCP backend with a knowledge graph, semantic search, and multi-tenant isolation. Free tier is 200 free credits (one credit per operation) activated by one click in the portal at [studiomeyer.io/portal/login](https://studiomeyer.io/portal/login), no card required.

Each template is multi-provider out of the box. Pick OpenAI (default `gpt-5.4-mini`, May 2026) or Anthropic (`claude-haiku-4-5`) with one click. Add a third branch for Gemini or local Ollama by adding one Switch rule. Memory writes stay identical regardless of which LLM you picked.

## Quick Start

```bash
# 1. Clone or fetch a single workflow
git clone https://github.com/studiomeyer-io/n8n-templates.git

# 2. In your n8n instance, install the StudioMeyer Memory community node
npm install n8n-nodes-studiomeyer-memory
# (or via UI: Settings → Community Nodes → Install: n8n-nodes-studiomeyer-memory)

# 3. Get a free API key
open https://memory.studiomeyer.io/dashboard/keys

# 4. Pick a template, copy its workflow.json, import into n8n, fill the SET-ME markers, activate.
```

Detailed walkthrough per template lives inside each `templates/NN-slug/README.md`.

## Templates

### Tier 1 · Maximum ROI · Hardened (v0.4.0)

All three ship the four opt-in production patterns (HMAC verify, rate limit, idempotency, error branches) as actual nodes plus an always-on error branch. T02 + T03 are live-verified against `memory.studiomeyer.io`. T01 awaits an end-to-end Vapi/Retell smoke trace before the distribution push. See [STATUS.md](./STATUS.md) for ground truth.

| # | Template | Trigger | Memory pattern | LLM | Status |
|---|---|---|---|---|---|
| 1 | [Voice Agent Cross-Session Memory](./templates/01-voice-agent-cross-session-memory/) | Vapi / Retell webhook | entity.search → entity.observe | Multi-provider | Hardened, E2E-pending |
| 2 | [AI Customer Support with History](./templates/02-customer-support-with-history/) | Telegram (swappable for WhatsApp / Slack) | entity.search → entity.open dossier | Multi-provider | Hardened, live-verified |
| 3 | [Personal Assistant Long-Term Memory](./templates/03-personal-assistant-long-term-memory/) | Telegram with intent classifier | memory.search → memory.synthesize | Multi-provider | Hardened, live-verified |

### Tier 2 · Reference cases · Hardened (v0.5.0-prep)

All three ship the same four production patterns, structurally validated against n8n 2.15.0 (live-import + pre-activation-check passed). End-to-end smoke against the real backend per template (Telegram bot, web-chat widget, Pipedrive instance) is the next pass.

| # | Template | Trigger | Memory pattern | LLM | Status |
|---|---|---|---|---|---|
| 4 | [Restaurant Stammgast-Bot](./templates/04-restaurant-stammgast-bot/) | Telegram (phone-aware) | entity.search → entity.observe | Multi-provider | Hardened, E2E-pending |
| 5 | [Tourist-Bot Repeat-Visitor](./templates/05-tourist-bot-repeat-visitor/) | Generic web-chat webhook | entity.search → entity.observe | Multi-provider | Hardened, E2E-pending |
| 6 | [Lead-Qualifier with BANT+I and Pipedrive](./templates/06-lead-qualifier-pipedrive/) | Form webhook | entity.search → entity.observe → Pipedrive create-deal | Multi-provider | Hardened, E2E-pending |

T04 references [MenuFlow](https://studiomeyer.io/services/tourismus/menuflow) as a live reference deployment. T05 references [MallorcaBot](https://mallorcabot.de). T06 cross-sells to [StudioMeyer CRM](https://crm.studiomeyer.io) with Pipedrive as the integration target.

### Tier 3 · Niche differentiators · Hardened (v0.5.0-prep)

| # | Template | Trigger | Memory pattern | LLM | Status |
|---|---|---|---|---|---|
| 7 | [Meeting-Bot Cross-Meeting Continuity](./templates/07-meeting-bot-cross-meeting-continuity/) | Fathom / Otter / Granola webhook | entity.search by participant-set hash → memory.synthesize | Multi-provider | Hardened, E2E-pending |
| 8 | [Mem0 / Zep Migration to StudioMeyer Memory](./templates/08-mem0-zep-migration/) | Manual or HTTP Trigger | entity.create + observe + learn (batch ETL) | None (pure ETL) | Developer-preview |

T07 is the participant-set-hash pattern, no other Memory backend ships cross-meeting continuity as a first-class API. T08 is the migration whitespace (Zep deprecated their Community Edition in late 2025, Mem0 has no migration tool, no third-party utility exists).

## Architecture

The shared backbone across every template:

```
[Trigger]
    │
    ▼
[Normalize Payload]              ← code node parses the provider-specific shape
    │
    ▼
[Memory: Lookup]                 ← entity.search / memory.search depending on use case
    │
    ▼
   ┌──┤ Decision (existing?) ├──┐
   ▼ yes                      no ▼
[Memory: Read context]      [Memory: Create entity]
   │                           │
   └────────────┬──────────────┘
                ▼
[Build Prompt]                   ← injects retrieved context into the LLM system prompt
                │
                ▼
[Set Provider] → [Route] ─┬─ [OpenAI] ─┐
                          └─ [Anthropic] ─┘
                                    ▼
                          [Normalize LLM Output]
                                    ▼
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
        [Reply to user]  [Memory: Observe]  [Memory: Learn]
```

The Memory operations on the left are provided by the [n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory) community node. Sixteen operations across four resources (Memory, Entity, Session, Insight). Same auth, same tenant scope, free tier covers all of Tier 1 for a small business.

The LLM branches converge into a single Code node that extracts `replyText` from either provider's response shape. The downstream nodes (reply + memory writes) never know which LLM answered.

## Prerequisites

Every template needs:

1. **n8n instance with the CVE-2026-27493 fix applied.** That means **>= 2.9.3** on the 2.x stable channel (the default n8n.io ships), **>= 2.10.1** on the 2.x latest/beta channel, or **>= 1.123.22** on the 1.x LTS channel. CVE-2026-27493 is an unauthenticated RCE in Form nodes (CVSS 9.5, fixed Feb 2026). None of these templates use Form nodes themselves, but you should not run a vulnerable n8n in any case. Cloud, self-hosted, and Docker installs all qualify as long as the version floor is met.
2. **The community node** installed: `npm install n8n-nodes-studiomeyer-memory` for self-hosted, or via *Settings → Community Nodes* on n8n Cloud.
3. **A free API key** from [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Add as credential `StudioMeyer Memory API`.
4. **An LLM credential** in n8n. OpenAI (default for these templates) or Anthropic. Free tiers from both providers cover the Tier 1 workloads.
5. **Provider-specific credentials** if your trigger needs them: Telegram bot token, Vapi / Retell webhook URL, etc. Documented per template.

## What makes these templates different

Most n8n templates on the marketplace are demos. They show the happy path and stop. We do two things differently:

**Built into every template's workflow.json (verifiable when you import):**

| Pattern | Why it matters | Where it lives |
|---|---|---|
| **Multi-provider LLM switch** | Provider lock-in is bad for the builder. They want to A/B OpenAI vs Anthropic, swap if costs spike, fall back if one is down. | `Set Provider` + `Route by Provider` nodes let the builder pick OpenAI (default `gpt-5.4-mini`) or Anthropic (`claude-haiku-4-5`). Both branches converge in a `Normalize LLM Output` Code node that flattens the provider-specific response shape. |
| **Memory de-duplication** | Same observation written twice creates noise in the knowledge graph. | StudioMeyer Memory's gatekeeper deduplicates on >95% content similarity automatically. Our templates rely on it. The `action: SKIPPED, similarity: 1` response on retries is the verification signal. |

**Three of these patterns ship as opt-in nodes in Templates 02 and 03, all four ship as opt-in nodes in Template 01 (gated by env vars, default-off so the import boots clean). Toggle via env var to enable for production:**

| Pattern | Why it matters | What you do |
|---|---|---|
| **Idempotency** | Trigger providers retry on 5xx. Without dedup, every retry creates a duplicate memory entry and a duplicate LLM bill. | Add a Code node at the top of `Normalize Payload` with `$getWorkflowStaticData` 5-minute dedup window keyed on the provider's call/update id. Swap to Redis SET NX for clustered deployments. |
| **Error branches** | LLM 429 / 500 / timeouts happen. Without an error branch, the workflow silently fails and the user gets nothing. | Set "On Error: Continue (using error output)" on each LLM Reply node, route the red error pin to a Code node that produces a graceful fallback reply and writes a `category: mistake` learning to Memory. Inline error pin uses `{{ $json.error.message }}`. The other documented syntax `{{ $json.execution.error.message }}` is only for a separate Error Trigger Workflow. The often-quoted `{{ $error.message }}` does not exist in n8n. |
| **HMAC webhook verification** | Public webhooks without signature verification can be hit by anyone. At LLM scale this is a $1000 bill in 5 minutes. | Add a Code node right after the Webhook trigger that verifies the provider signature (Vapi `x-vapi-signature`, Retell `x-retell-signature`, Telegram secret-token) with `crypto.timingSafeEqual` and rejects unsigned requests. |
| **Rate limiting** | Same reason as HMAC. Even with HMAC, a stolen key needs throttling. | Per-IP token bucket Code node, 60 requests / 5 min default. Tracked in `$getWorkflowStaticData('global').rateBuckets`. Swap to Redis `INCR + EXPIRE` for clusters. |

The first table is what you get out of the box. The second table is what you wire in when you move from dev to production. Both are documented end-to-end so neither has to be re-invented per template.

v0.4.0 (live as of S950, 2026-05-01) shipped the four production-patterns as opt-in nodes inside every Tier-1 + Tier-2 + Tier-3 workflow.json, gated by env vars (default-off, pass-through when unset). Sprint B + C (T04-T08, S955) extend the same pattern to all 5 new templates. So the patterns are wired and one env-var-toggle away rather than one paste away.

## How we compare to other public n8n template repos

We audited five high-star public n8n template / workflow repos in April 2026 ([awesome-n8n-templates](https://github.com/enescingoz/awesome-n8n-templates), [n8n-workflows](https://github.com/Zie619/n8n-workflows), and three others) plus a sample of [n8n.io/workflows](https://n8n.io/workflows) listings. None of them ship the production patterns above. Most stop at the happy path:

| Capability | Most public n8n template repos | This repo |
|---|---|---|
| Workflow runs once you import | yes | yes |
| Sticky notes | sometimes | always (every SET-ME marker) |
| Cover image | sometimes | always (1200x630, suite-consistent) |
| Multi-provider LLM switch | rare | yes (Set Provider + Switch + Normalize Output) |
| Idempotency pattern | none we found | documented snippet, opt-in node |
| Error-output branches with correct n8n syntax | none we found | documented snippet, opt-in node |
| Webhook HMAC verification | none we found | documented snippet (Vapi / Retell / Telegram / Stripe), opt-in env-var |
| Rate limiting | none we found | documented snippet, opt-in node |
| Memory layer with knowledge graph | n/a (most templates are stateless) | StudioMeyer Memory built in |
| Hard n8n minVersion floor with CVE awareness | rare | declared, CVE-2026-27493 cited |
| MIT license | usually | yes |
| Open governance (CONTRIBUTING + COC + SECURITY + ECOSYSTEM) | rare | yes |
| Repo CI that validates workflows | rare | GitHub Actions, blocks merges on broken refs / em-dashes / live credentials |

The middle four rows are wired as actual opt-in nodes in `workflow.json` across all eight Tier-1+2 templates (HMAC verify, rate limit, idempotency with `Skip If Duplicate` IF gateway plus `Respond Duplicate` respondToWebhook on webhook-trigger templates, error branches), gated by env vars and pass-through by default.

## FAQ

**Do I need StudioMeyer Memory to use these templates?** Two of them yes, one less so. Templates 01 and 02 are built around the memory loop (entity.search → reason → write back). Template 03 mostly uses memory.search and memory.learn. Strip the Memory nodes from Template 02 and you have a generic Telegram support bot without history. We won't pretend that gives you the full value, but the workflow.json is yours to fork.

**Why two LLM providers?** Provider lock-in is bad for the builder. OpenAI rate-limits during product launches, Anthropic has had outages, Gemini hallucinates differently than both. The `Set Provider` node lets you swap with one click. We pick OpenAI as default because that's the bigger audience.

**Why a hard n8n version floor?** [CVE-2026-27493](https://nvd.nist.gov/vuln/detail/CVE-2026-27493) (CVSS 9.5) is an unauthenticated RCE in Form nodes, fixed Feb 2026. The patch is in **2.9.3 (stable channel)**, **2.10.1 (latest channel)**, and **1.123.22 (LTS channel)**. The README badge shows 2.10.1+ as the simplest single-line ask, but any of the three patched-version-or-newer combinations works. We do not use Form nodes ourselves, but you should not run a vulnerable n8n in any case.

**Can I use this with n8n Cloud?** Yes. All templates run unchanged on n8n Cloud, n8n Self-Hosted, n8n Docker, and the n8n Desktop app. The community node `n8n-nodes-studiomeyer-memory` installs via *Settings → Community Nodes* on Cloud. Webhook trigger URLs are auto-generated by n8n.

**What's the cost per execution?** Roughly $0.002 to $0.007 depending on which template, provider, and reply length. Detailed cost tables in each template's README. The free Memory tier (200 credits, one credit per operation) plus OpenAI / Anthropic free credits covers an evaluation run; for production volumes Pro at EUR 29/mo lifts the cap.

**Are these production-ready?** Honest answer: production-pattern hardened in v0.4.0-prep, not a one-click production deploy yet. **All three Tier-1 templates** ship the four opt-in production patterns (idempotency, error branches, webhook security, rate limit) as actual nodes in `workflow.json`, gated by env vars and pass-through by default. Memory de-dup is server-side. Template 01 (Voice Agent) is at the same code-level hardening as 02 + 03 but is the only one without an end-to-end live smoke test against a real Vapi or Retell account yet, which is the gating item for the v0.4.0 final release and the distribution push. See [STATUS.md](./STATUS.md) for the per-template ground truth and [PRODUCTION_CHECKLIST.md](./PRODUCTION_CHECKLIST.md) for the env vars + signing secrets + Node-builtin allowlist + monitoring you need before flipping these to production. CI blocks workflow.json files with em-dashes, missing references, the n8n-API-rejected `meta`/`staticData`/`versionId`/`id`/`tags` keys, and obvious credential leaks (literal API keys, Bearer tokens, JWTs). All three templates promote to "production-ready" in the same release cycle when Template 01 ships its E2E voice smoke trace.

**How do I contribute?** Open a [template request issue](https://github.com/studiomeyer-io/n8n-templates/issues/new?template=template_request.md) so we can confirm scope. Then copy `templates/_TEMPLATE/`, fill it in, smoke-test in your own n8n, open a PR. The [CONTRIBUTING.md](./CONTRIBUTING.md) covers the full bar.

**Why is the workflow.json so verbose?** Sticky notes. The yellow notes mark every SET-ME spot for the importing builder. n8n's own template-marketplace creator-hub flags missing sticky notes as the #1 rejection reason for new submissions. We over-comment on purpose.

**Where do I report a security issue?** [SECURITY.md](./SECURITY.md). Email `hello@studiomeyer.io` with subject `[security] n8n-templates`. We aim for 48-hour acknowledgement and a 7-day patch on high-severity issues.

## Memory-free variant

If your workflow does not need cross-session memory (the bot does not need to remember who called yesterday, the support agent does not need prior tickets, the digest is computed fresh each run), use the sister repo:

**[studiomeyer-io/n8n-workflows](https://github.com/studiomeyer-io/n8n-workflows)** ships **15 production templates** (v0.3.1) with the same four opt-in patterns this repo ships (HMAC verify, rate limit, idempotency with `Skip If Duplicate` IF gateway plus `Respond Duplicate` respondToWebhook on webhook-trigger templates, error branches), but no Memory dependency: Form to CRM Lead Router with BANT scoring, Stripe Lifecycle to Slack with timestamped HMAC, Uptime Monitor with state-change-only alerting, SSL Certificate Expiry Watcher, Slack Channel Daily Digest, Calendly to CRM Sync, GitHub Issues Router, RSS to Multi-Channel Social, Calendar Conflict Detector, CSV Bulk Validator, Email to Notion, Postgres to Sheets Sync, Webhook Audit Trail with advisory-locked hash chain, Telegram Translator Bot, YouTube Channel to Notion.

Both repos share the **same CI guards** (workflow validation, em-dash guard, forbidden-keys check, credential-leak scan, `$input.first()` semantic lint, idempotency-skipped flow guard), the same hard n8n version floor, the same brand-consistent cover-image standard. The split is deliberate: this repo focuses on what changes when you add Memory, the sister repo focuses on production patterns that work for builders who do not need memory yet.

## Distribution status

This repo lives on GitHub. Submission to the wider n8n distribution channels happens incrementally as templates mature. Current state:

| Channel | Status |
|---|---|
| GitHub repo (this) | live since v0.1.0 |
| GitHub Discussions enabled | yes |
| GitHub Topics (n8n, n8n-templates, ai-agents, mcp, ...) | set |
| GitHub Social Preview image | set |
| n8n.io/workflows submissions | not yet, planned post-v0.4.0 |
| awesome-n8n-templates PR | not yet, planned post-v0.4.0 |
| n8n Discord show-and-tell | not yet |
| dev.to long-form tutorial per template | not yet |
| Reddit r/n8n + r/AI_Agents posts | not yet |
| LinkedIn DACH PDF carousels | not yet |

All eight Tier-1+2 templates ship the four opt-in production patterns as actual nodes in `workflow.json` (v0.4.0-prep). The reason we hold submissions back is end-to-end smoke against real provider backends: Template 01 (Voice Agent) needs a Vapi or Retell trial-account run before the first distribution push.

## Repo structure

```
n8n-templates/
├── README.md                       # this file
├── STATUS.md                       # per-template ground truth (hardened / pending)
├── PRODUCTION_CHECKLIST.md         # env vars + secret tokens + monitoring before launch
├── ECOSYSTEM.md                    # the rest of the StudioMeyer toolkit
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE                         # MIT
├── .github/
│   ├── FUNDING.yml
│   ├── ISSUE_TEMPLATE/             # bug + template-request
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── workflows/                  # CI: workflow validation, em-dash guard, code-node semantic lint, idempotency-skipped flow guard, smoke-test stub
├── scripts/                        # idempotent auto-fix helpers (input-access, idempotency-respond)
├── examples/                       # sample provider payloads for smoke-testing
│   ├── README.md                   # how to use the payloads (curl recipes)
│   ├── vapi-end-of-call.json
│   ├── retell-call-ended.json
│   └── telegram-message.json
└── templates/
    ├── _TEMPLATE/                  # skeleton for new contributions
    ├── 01-voice-agent-cross-session-memory/             # Vapi/Retell webhook + memory loop
    ├── 02-customer-support-with-history/                # Telegram + CRM history
    ├── 03-personal-assistant-long-term-memory/          # Telegram + cross-session recall
    ├── 04-restaurant-stammgast-bot/                     # Telegram + repeat-guest pattern
    ├── 05-tourist-bot-repeat-visitor/                   # web webhook + visitor session
    ├── 06-lead-qualifier-pipedrive/                     # form webhook + BANT-I + Pipedrive
    ├── 07-meeting-bot-cross-meeting-continuity/         # Fathom/Otter/Granola + Slack
    └── 08-mem0-zep-migration/                           # ETL out of Mem0 / Zep
```

Each template folder is self-contained. Copy any one of them out of this repo and it still works.

## How to import a template

1. Open the template folder in this repo (for example `templates/01-voice-agent-cross-session-memory/`).
2. Open `workflow.json`. Copy the full contents.
3. In n8n: top-right menu → *Import from clipboard* → paste → *Import*.
4. Open the imported workflow. Yellow sticky notes marked **`>> SET ME <<`** flag every spot you need to configure (webhook URLs, API keys, provider names).
5. Activate when the markers are filled.

The per-template README explains data flow, memory keys, and extension recipes.

## Quality Gate

Every template in this repo is held against an internal quality standard:

- 12 mandatory README sections (intro, data flow, architecture diagram, memory model table, setup, multi-provider, extending, cost, gotchas, related templates, footer)
- Multi-provider LLM switch when an LLM call is involved
- No em-dashes (LLM-content signature, downranked by indexers)
- No real credentials in the committed `workflow.json`
- A live n8n smoke test before merge
- A Flux-generated cover image (`cover.png`)
- A 3-agent code review (analyst + critic + research) on substantial changes

CI enforces the structural pieces (workflow.json validity, no em-dashes, no forbidden top-level keys, no credential leaks). The editorial pieces (tone, sticky-note clarity, naming) are reviewed by maintainers per PR.

## Versioning

Repo follows [Semantic Versioning](https://semver.org/). PATCH for bug fixes in templates. MINOR for new templates or feature additions. MAJOR for breaking changes (renamed nodes, removed parameters, changed memory schema).

Tags are pushed for every MINOR and MAJOR release. See [CHANGELOG.md](./CHANGELOG.md).

## Contributing

We welcome new templates that solve a real workflow problem. The process:

1. Read [CONTRIBUTING.md](./CONTRIBUTING.md) for the bar.
2. Open a [template request issue](https://github.com/studiomeyer-io/n8n-templates/issues/new?template=template_request.md) so we can confirm scope before you build.
3. Copy `templates/_TEMPLATE/` to a new folder, fill it in, smoke-test in your own n8n instance, open a PR.

The PR template includes the per-template checklist. Reviewers verify structure first, content second.

## Why trust the Memory backend

Every template in this repo depends on the [n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory) community node and the hosted [memory.studiomeyer.io](https://memory.studiomeyer.io) service. Before you wire either into a production workflow, here is what you can verify externally:

| Trust signal | Evidence |
|---|---|
| **Custom node on npm with provenance** | [n8n-nodes-studiomeyer-memory](https://www.npmjs.com/package/n8n-nodes-studiomeyer-memory) · published with `--provenance` via GitHub Actions OIDC, shasum verifiable on npm's site |
| **Custom node source + changelog** | [github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory) · MIT, 58 vitest tests covering tool-call mapping + SSRF guard + result parsing, agent-code-review hardening passes documented in CHANGELOG |
| **Security policy** | [SECURITY.md](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory/blob/main/README.md) on the node repo · 48-hour acknowledgement target, 7-day patch on high-severity |
| **EU hosting + DSGVO** | Supabase Frankfurt (eu-central-1) · multi-tenant with App-Layer tenant_id isolation + Defense-in-Depth `rowsecurity=true` on every table · Hetzner Germany infra |
| **Data export + delete** | Account holders can export full memory contents via the dashboard at [studiomeyer.io/portal/memory](https://studiomeyer.io/portal/memory) and trigger a hard-delete of the tenant via the same UI · DSGVO Art. 17 erasure path |
| **Bi-temporal model** | Every `Learning` and `Decision` carries `valid_from` + `valid_to` plus a confidence score that decays over time. Stale facts fade automatically, contradictions are surfaced via `nex_contradictions` |
| **Multi-tenant isolation** | Static-analysis CI guard `tenant-isolation-static.test.ts` blocks raw SQL without `tenant_id` filter at PR time · documented in [docs/claude/nex-memory-system.md](https://github.com/studiomeyer-io/) |
| **API-key rotation** | Generate, list, and revoke keys at any time via the dashboard. Optional HMAC-SHA256 pepper (`NEX_API_KEY_PEPPER`) for self-hosted server defense-in-depth |
| **Auth modes** | API Key (paste from dashboard) or OAuth 2.1 with PKCE access token. Discovery doc at [memory.studiomeyer.io/.well-known/oauth-authorization-server](https://memory.studiomeyer.io/.well-known/oauth-authorization-server) |
| **Open source posture** | The custom node and these templates are MIT-licensed (use anywhere, commercial OK). The Memory **server** is hosted SaaS, not currently self-hostable. If self-host is a hard requirement for you, talk to us about a license. |

If any of those trust signals do not meet your bar, do not wire these templates into production. The point of this section is that you can verify each one yourself in five minutes with no NDA.

## Related projects

- **Memory-free variant:** [github.com/studiomeyer-io/n8n-workflows](https://github.com/studiomeyer-io/n8n-workflows) (5 production templates without Memory dependency)
- **Custom node source:** [github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory)
- **Memory product:** [memory.studiomeyer.io](https://memory.studiomeyer.io) · [marketing page](https://studiomeyer.io/services/memory)
- **Memory live demo:** [studiomeyer.io/services/memory/demo](https://studiomeyer.io/services/memory/demo) (interactive 3D knowledge graph)
- **Long-form tutorials:** [studiomeyer.io/blog](https://studiomeyer.io/blog) (filter: `n8n`)
- **Full ecosystem:** [ECOSYSTEM.md](./ECOSYSTEM.md)

## About StudioMeyer

[StudioMeyer](https://studiomeyer.io) is an AI and design studio based in Palma de Mallorca, working with clients worldwide. We build custom websites and AI infrastructure for small and medium businesses. Production stack on Claude Agent SDK, MCP and n8n, with Sentry, Langfuse and LangGraph for observability and an in-house guard layer.

## License

MIT, see [LICENSE](./LICENSE). Use these templates anywhere, including commercial deployments. Attribution appreciated, not required.

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues + ideas at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues). The reason a search query like "n8n voice agent memory" surfaces this repo is craft on every template, not luck.*
