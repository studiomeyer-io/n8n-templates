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

Most n8n templates on the marketplace are demos. They show the happy path and stop. We test ours against real production patterns that other public templates miss:

| Pattern | Why it matters | Where in our templates |
|---|---|---|
| **Idempotency** | Trigger providers retry on 5xx. Without dedup, every retry creates a duplicate memory entry and a duplicate LLM bill. | `Normalize Payload` Code node uses `$getWorkflowStaticData` for an in-memory dedup window. Swap to Redis SET NX for clustered deployments. |
| **Error branches** | LLM 429 / 500 / timeouts happen. Without an error branch, the workflow silently fails and the user gets nothing. | LLM Reply nodes have "On Error: Continue (Error Output)" set. Error path falls back to a graceful reply and logs the failure to Memory. |
| **HMAC webhook verification** | Public webhooks without signature verification can be hit by anyone. At LLM scale this is a $1000 bill in 5 minutes. | First Code node after the webhook verifies provider HMAC (Vapi `x-vapi-signature`, Retell `x-retell-signature`, Telegram secret-token). Off by default, opt-in via env var. |
| **Rate limiting** | Same reason as HMAC. Even with HMAC, a stolen key needs throttling. | Per-IP 60-requests-per-5-min window in `Normalize Payload`. Configurable. |
| **Multi-provider switch** | Provider lock-in is bad for the builder. They want to A/B OpenAI vs Anthropic, swap if costs spike, fall back if one is down. | `Set Provider` + `Route by Provider` switch lets the builder pick OpenAI (default) or Anthropic. Both branches converge in a Code node that normalizes the response shape. |
| **Memory de-duplication** | Same observation written twice creates noise. | StudioMeyer Memory's gatekeeper deduplicates on >95% similarity automatically. Our templates rely on it. |

These five patterns are the difference between a template you import to learn and a template you import to ship.

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
