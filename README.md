<!-- studiomeyer-mcp-stack-banner:start -->
> **Part of the [StudioMeyer MCP Stack](https://studiomeyer.io)**, Built in Mallorca · ⭐ if you use it
<!-- studiomeyer-mcp-stack-banner:end -->

<div align="center">

# n8n Templates · StudioMeyer Memory

**Drop-in n8n workflows that turn AI agents from amnesia patients into systems that remember.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![n8n compatible](https://img.shields.io/badge/n8n-1.50%2B-FF6E5C.svg)](https://n8n.io)
[![Custom Node](https://img.shields.io/npm/v/n8n-nodes-studiomeyer-memory?label=community%20node&color=blue)](https://www.npmjs.com/package/n8n-nodes-studiomeyer-memory)
[![Templates](https://img.shields.io/badge/templates-3%20live-brightgreen.svg)](#templates)
[![Memory backend](https://img.shields.io/badge/memory-studiomeyer.io-d4af37.svg)](https://memory.studiomeyer.io)
[![CI](https://github.com/studiomeyer-io/n8n-templates/actions/workflows/validate-workflows.yml/badge.svg)](https://github.com/studiomeyer-io/n8n-templates/actions)

Voice agents · customer support · personal assistants · cross-session memory · multi-provider LLM

[Quick Start](#quick-start) · [Templates](#templates) · [Architecture](#architecture) · [Brand Bibel](./N8N-BRAND-BIBEL.md) · [Ecosystem](./ECOSYSTEM.md) · [Contributing](./CONTRIBUTING.md)

</div>

---

## Why this exists

Without persistent memory every AI interaction starts from zero. Voice agents forget callers. Support bots ask returning customers their email three times. Personal assistants miss the project context you discussed yesterday.

These templates fix that. Each workflow follows the same loop: **search the memory, reason with the context, write the outcome back**. Memory is provided by [StudioMeyer Memory](https://memory.studiomeyer.io), a hosted MCP backend with a knowledge graph, semantic search, and multi-tenant isolation. Free tier covers about 10 000 operations per month.

Each template is multi-provider out of the box. Pick OpenAI (default `gpt-5-mini`) or Anthropic (`claude-haiku-4-5`) with one click. Add a third branch for Gemini or local Ollama by adding one Switch rule. Memory writes stay identical regardless of which LLM you picked.

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

### Tier 1 · Maximum ROI · Live

| # | Template | Trigger | Memory pattern | LLM |
|---|---|---|---|---|
| 1 | [Voice Agent Cross-Session Memory](./templates/01-voice-agent-cross-session-memory/) | Vapi / Retell webhook | entity.search → entity.observe | Multi-provider |
| 2 | [AI Customer Support with History](./templates/02-customer-support-with-history/) | Telegram (swappable for WhatsApp / Slack) | entity.search → entity.open dossier | Multi-provider |
| 3 | [Personal Assistant Long-Term Memory](./templates/03-personal-assistant-long-term-memory/) | Telegram with intent classifier | memory.search → memory.synthesize | Multi-provider |

### Tier 2 · Reference cases · Coming next

- **Restaurant Stammgast Bot**, Telegram + entity tracking, backed by [MenuFlow](https://menuflow.studiomeyer.io) as a live reference deployment.
- **Tourist Bot with Repeat-Visitor Recognition**, Web chat + session memory, backed by [MallorcaBot](https://mallorcabot.de).
- **Lead Qualifier with Conversation Memory**, Form trigger + BANT discovery, cross-sell to [StudioMeyer CRM](https://crm.studiomeyer.io).

### Tier 3 · Niche differentiators · Backlog

- **Meeting Bot with Context Continuity**, Otter / Fathom transcripts, multi-meeting synthesis.
- **Mem0 / Zep migration tool**, bulk-import existing memory backends into StudioMeyer Memory.

See [`N8N-BRAND-BIBEL.md`](./N8N-BRAND-BIBEL.md) for the full quality bar every template hits before merge.

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

1. **n8n instance at version 2.10.1 or higher** (Cloud, self-hosted, or Docker). 2.10.1 is the floor because of CVE-2026-27493 (unauthenticated RCE in Form nodes, fixed Feb 2026). 1.x users: upgrade to 1.123.22 or later. None of these templates use Form nodes themselves, but you should not run a vulnerable n8n in any case.
2. **The community node** installed: `npm install n8n-nodes-studiomeyer-memory` for self-hosted, or via *Settings → Community Nodes* on n8n Cloud.
3. **A free API key** from [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Add as credential `StudioMeyer Memory API`.
4. **An LLM credential** in n8n. OpenAI (default for these templates) or Anthropic. Free tiers from both providers cover the Tier 1 workloads.
5. **Provider-specific credentials** if your trigger needs them: Telegram bot token, Vapi / Retell webhook URL, etc. Documented per template.

## What makes these templates different

Most n8n templates on the marketplace are demos. They show the happy path and stop. We do two things differently:

**Built into every template's workflow.json (verifiable when you import):**

| Pattern | Why it matters | Where it lives |
|---|---|---|
| **Multi-provider LLM switch** | Provider lock-in is bad for the builder. They want to A/B OpenAI vs Anthropic, swap if costs spike, fall back if one is down. | `Set Provider` + `Route by Provider` nodes let the builder pick OpenAI (default `gpt-5-mini`) or Anthropic (`claude-haiku-4-5`). Both branches converge in a `Normalize LLM Output` Code node that flattens the provider-specific response shape. |
| **Memory de-duplication** | Same observation written twice creates noise in the knowledge graph. | StudioMeyer Memory's gatekeeper deduplicates on >95% content similarity automatically. Our templates rely on it. The `action: SKIPPED, similarity: 1` response on retries is the verification signal. |

**Documented as drop-in code-node snippets in the [N8N-BRAND-BIBEL.md](./N8N-BRAND-BIBEL.md), not yet wired into the default workflow.json (opt-in for production):**

| Pattern | Why it matters | What you do |
|---|---|---|
| **Idempotency** | Trigger providers retry on 5xx. Without dedup, every retry creates a duplicate memory entry and a duplicate LLM bill. | Add a Code node at the top of `Normalize Payload` with `$getWorkflowStaticData` 5-minute dedup window keyed on the provider's call/update id. Snippet in the Brand Bibel. Swap to Redis SET NX for clustered deployments. |
| **Error branches** | LLM 429 / 500 / timeouts happen. Without an error branch, the workflow silently fails and the user gets nothing. | Set "On Error: Continue (using error output)" on each LLM Reply node, route the red error pin to a Code node that produces a graceful fallback reply and writes a `category: mistake` learning to Memory. Inline error pin uses `{{ $json.error.message }}`. The other documented syntax `{{ $json.execution.error.message }}` is only for a separate Error Trigger Workflow. The often-quoted `{{ $error.message }}` does not exist in n8n. |
| **HMAC webhook verification** | Public webhooks without signature verification can be hit by anyone. At LLM scale this is a $1000 bill in 5 minutes. | Add a Code node right after the Webhook trigger that verifies the provider signature (Vapi `x-vapi-signature`, Retell `x-retell-signature`, Telegram secret-token) with `crypto.timingSafeEqual` and rejects unsigned requests. Snippet in the Brand Bibel. |
| **Rate limiting** | Same reason as HMAC. Even with HMAC, a stolen key needs throttling. | Per-IP token bucket Code node, 60 requests / 5 min default. Tracked in `$getWorkflowStaticData('global').rateBuckets`. Snippet in the Brand Bibel. Swap to Redis `INCR + EXPIRE` for clusters. |

The first table is what you get out of the box. The second table is what you wire in when you move from dev to production. Both are documented end-to-end so neither has to be re-invented per template.

Roadmap note: v0.4.0 will ship the four production-patterns as opt-in nodes inside the default workflow.json (off by default with sticky-note instructions to enable), so they are one click away rather than one paste away.

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

The middle four rows are the gap. We close them with snippet-level documentation today and node-level wiring in v0.4.0.

## FAQ

**Do I need StudioMeyer Memory to use these templates?** Two of them yes, one less so. Templates 01 and 02 are built around the memory loop (entity.search → reason → write back). Template 03 mostly uses memory.search and memory.learn. Strip the Memory nodes from Template 02 and you have a generic Telegram support bot without history. We won't pretend that gives you the full value, but the workflow.json is yours to fork.

**Why two LLM providers?** Provider lock-in is bad for the builder. OpenAI rate-limits during product launches, Anthropic has had outages, Gemini hallucinates differently than both. The `Set Provider` node lets you swap with one click. We pick OpenAI as default because that's the bigger audience.

**Why n8n 2.10.1 as the floor?** [CVE-2026-27493](https://nvd.nist.gov/vuln/detail/CVE-2026-27493) (CVSS 9.5) is an unauthenticated RCE in Form nodes, fixed Feb 2026. We do not use Form nodes, but you should not run a vulnerable n8n in any case.

**Can I use this with n8n Cloud?** Yes. All templates run unchanged on n8n Cloud, n8n Self-Hosted, n8n Docker, and the n8n Desktop app. The community node `n8n-nodes-studiomeyer-memory` installs via *Settings → Community Nodes* on Cloud. Webhook trigger URLs are auto-generated by n8n.

**What's the cost per execution?** Roughly $0.002 to $0.007 depending on which template, provider, and reply length. Detailed cost tables in each template's README. The free Memory tier (10k ops / month) plus OpenAI / Anthropic free credits cover several thousand executions before you spend anything.

**Are these production-ready?** The happy path is. The five production patterns (idempotency, error branches, HMAC, rate limiting, memory de-dup) are documented as drop-in code-node snippets in the [Brand Bibel](./N8N-BRAND-BIBEL.md) and you wire them in for hardened deployments. v0.4.0 will move the four opt-in patterns into the default workflow.json with sticky-note enable / disable instructions.

**How do I contribute?** Open a [template request issue](https://github.com/studiomeyer-io/n8n-templates/issues/new?template=template_request.md) so we can confirm scope. Then copy `templates/_TEMPLATE/`, fill it in, smoke-test in your own n8n, open a PR. The [CONTRIBUTING.md](./CONTRIBUTING.md) and [N8N-BRAND-BIBEL.md](./N8N-BRAND-BIBEL.md) cover the full bar.

**Why is the workflow.json so verbose?** Sticky notes. The yellow notes mark every SET-ME spot for the importing builder. n8n's own template-marketplace creator-hub flags missing sticky notes as the #1 rejection reason for new submissions. We over-comment on purpose.

**Where do I report a security issue?** [SECURITY.md](./SECURITY.md). Email `hello@studiomeyer.io` with subject `[security] n8n-templates`. We aim for 48-hour acknowledgement and a 7-day patch on high-severity issues.

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

The reason we hold submissions back is the production-patterns gap above. Templates with snippet-level docs are good. Templates with opt-in production nodes inside the workflow.json (v0.4.0) are great. We submit when great.

## Repo structure

```
n8n-templates/
├── README.md                       # this file
├── N8N-BRAND-BIBEL.md               # internal style guide for tone, structure, branding
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
│   └── workflows/                  # CI: workflow validation, em-dash guard
└── templates/
    ├── _TEMPLATE/                  # skeleton for new contributions
    ├── 01-voice-agent-cross-session-memory/
    │   ├── workflow.json
    │   ├── README.md
    │   ├── cover.md                # cover image spec
    │   └── cover.png               # 1200x630 cover
    ├── 02-customer-support-with-history/
    └── 03-personal-assistant-long-term-memory/
```

Each template folder is self-contained. Copy any one of them out of this repo and it still works.

## How to import a template

1. Open the template folder in this repo (for example `templates/01-voice-agent-cross-session-memory/`).
2. Open `workflow.json`. Copy the full contents.
3. In n8n: top-right menu → *Import from clipboard* → paste → *Import*.
4. Open the imported workflow. Yellow sticky notes marked **`>> SET ME <<`** flag every spot you need to configure (webhook URLs, API keys, provider names).
5. Activate when the markers are filled.

The per-template README explains data flow, memory keys, and extension recipes.

## Brand Bibel + Quality Gate

Every template in this repo is held against the rules in [`N8N-BRAND-BIBEL.md`](./N8N-BRAND-BIBEL.md):

- 12 mandatory README sections (intro, data flow, architecture diagram, memory model table, setup, multi-provider, extending, cost, gotchas, related templates, footer)
- Multi-provider LLM switch when an LLM call is involved
- No em-dashes (LLM-content signature, downranked by indexers)
- No real credentials in the committed `workflow.json`
- A live n8n smoke test before merge
- A Flux-generated cover image (`cover.png`)
- A 3-agent code review (analyst + critic + research) on substantial changes

CI enforces the structural pieces. The brand bibel covers the editorial pieces.

## Versioning

Repo follows [Semantic Versioning](https://semver.org/). PATCH for bug fixes in templates. MINOR for new templates or feature additions. MAJOR for breaking changes (renamed nodes, removed parameters, changed memory schema).

Tags are pushed for every MINOR and MAJOR release. See [CHANGELOG.md](./CHANGELOG.md).

## Contributing

We welcome new templates that solve a real workflow problem. The process:

1. Read [`N8N-BRAND-BIBEL.md`](./N8N-BRAND-BIBEL.md) for the bar.
2. Open a [template request issue](https://github.com/studiomeyer-io/n8n-templates/issues/new?template=template_request.md) so we can confirm scope before you build.
3. Copy `templates/_TEMPLATE/` to a new folder, fill it in, smoke-test in your own n8n instance, open a PR.

The PR template includes the brand-bibel checklist. Reviewers verify structure first, content second.

## Related projects

- **Custom node source:** [github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory)
- **Memory product:** [memory.studiomeyer.io](https://memory.studiomeyer.io) · [marketing page](https://studiomeyer.io/services/memory)
- **Memory live demo:** [studiomeyer.io/services/memory/demo](https://studiomeyer.io/services/memory/demo) (interactive 3D knowledge graph)
- **Long-form tutorials:** [studiomeyer.io/blog](https://studiomeyer.io/blog) (filter: `n8n`)
- **Full ecosystem:** [ECOSYSTEM.md](./ECOSYSTEM.md)

## License

MIT, see [LICENSE](./LICENSE). Use these templates anywhere, including commercial deployments. Attribution appreciated, not required.

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues + ideas at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues). The reason a search query like "n8n voice agent memory" surfaces this repo is the work in [N8N-BRAND-BIBEL.md](./N8N-BRAND-BIBEL.md), not luck.*
