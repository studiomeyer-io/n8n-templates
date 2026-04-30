# n8n Templates — StudioMeyer Memory

Production-ready n8n workflow templates that turn StudioMeyer Memory into the long-term memory layer for voice agents, support bots, and personal assistants.

> **What is StudioMeyer Memory?** A managed AI memory backend with a knowledge graph, semantic search, entity tracking, and session continuity. Hosted at [memory.studiomeyer.io](https://memory.studiomeyer.io) and accessible from n8n via the official community node [`n8n-nodes-studiomeyer-memory`](https://www.npmjs.com/package/n8n-nodes-studiomeyer-memory).

## Why memory matters in n8n

Without persistent memory, every AI interaction starts from zero. Voice agents forget callers. Support bots ask returning customers their email three times. Personal assistants miss the project context you discussed yesterday.

These templates fix that. Each one is:

- **Drop-in ready** — import the JSON, fill in two credentials, run.
- **Memory-first** — the core loop is search → reason → learn, repeated.
- **Production-aware** — error branches, idempotency keys, rate-limit-friendly.

## Templates

### Tier 1 — Maximum ROI

| # | Template | Stack | Best for |
|---|---|---|---|
| 1 | [Voice Agent Cross-Session Memory](./templates/01-voice-agent-cross-session-memory/) | Vapi/Retell webhook → Memory.search → Claude → Memory.learn | Voice-AI builders. Caller is recognized across calls. |
| 2 | [AI Customer Support with Customer History](./templates/02-customer-support-with-history/) | WhatsApp / Telegram → Memory.entity-search → Claude → Memory.learn | Mid-market + agencies running customer-facing bots. |
| 3 | [Personal Assistant with Long-Term Memory](./templates/03-personal-assistant-long-term-memory/) | Telegram → Memory.search → Tool-Use Agent (Calendar, Gmail, Notion) → Memory.learn | Indie hackers + solo founders. |

### Tier 2 — Reference cases (coming soon)

- Restaurant Stammgast Bot (Telegram + entity tracking) — backed by [MenuFlow](https://menuflow.studiomeyer.io)
- Tourist Bot with Repeat-Visitor Recognition — backed by [MallorcaBot](https://mallorcabot.de)
- Lead Qualifier with Conversation Memory — Cross-sell to [StudioMeyer CRM](https://crm.studiomeyer.io)

### Tier 3 — Niche differentiators (coming soon)

- Meeting Bot with Context Continuity (Otter / Fathom)
- AI-Agent Memory Migration (Mem0 / Zep → StudioMeyer)

## Prerequisites

Every template needs:

1. **n8n instance** ≥ 1.50 (Cloud, self-hosted, or Docker — all fine).
2. **StudioMeyer Memory community node** installed:
   ```bash
   # On self-hosted n8n
   npm install n8n-nodes-studiomeyer-memory
   # Or on n8n Cloud / hosted: Settings → Community Nodes → Install
   #   Package name: n8n-nodes-studiomeyer-memory
   ```
3. **API key** from [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). The free tier covers ~10k operations/month, enough to run all three Tier 1 workflows for a small business.
4. **Credential setup in n8n** — once per instance:
   - Credentials → New → search "StudioMeyer Memory API"
   - Auth Mode: API Key
   - API Key: paste your key
   - Save. Test should return `OK`.

## How to import a template

1. Open the template folder in this repo (e.g. `templates/01-voice-agent-cross-session-memory/`).
2. Open `workflow.json`. Copy the entire contents.
3. In n8n: top-right menu → Import from clipboard → paste → Import.
4. Open the imported workflow. The yellow **`>>` SET ME` <<`** notes mark every place you need to plug in your own webhook URLs, API keys, or model choice.
5. Activate the workflow when ready.

Each template ships with its own README that explains the data flow, the memory keys used, and how to extend it.

## Repo structure

```
n8n-templates/
├── README.md                  # this file
├── LICENSE                    # MIT
├── CONTRIBUTING.md            # how to add a new template
└── templates/
    ├── 01-voice-agent-cross-session-memory/
    │   ├── workflow.json      # importable n8n workflow
    │   ├── README.md          # what it does, how to set up, how to extend
    │   └── cover.md           # cover-image spec for n8n.io submission
    ├── 02-customer-support-with-history/
    └── 03-personal-assistant-long-term-memory/
```

## Versioning

Each template's `workflow.json` is pinned to the n8n schema version it was authored against (visible in the file's `meta.templateCredsSetupCompleted` and node `typeVersion` fields). We test against the latest stable n8n release and update templates when n8n introduces breaking node-API changes.

## Contributing

We welcome new templates that solve a real workflow problem with memory at the center. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the structure we expect and the review checklist.

## Related

- **Custom node source:** [github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory)
- **Memory product page:** [studiomeyer.io/services/memory](https://studiomeyer.io/services/memory)
- **Memory live demo:** [studiomeyer.io/services/memory/demo](https://studiomeyer.io/services/memory/demo)
- **Long-form tutorials:** [studiomeyer.io/blog](https://studiomeyer.io/blog) (filter: n8n)

## License

MIT — see [LICENSE](./LICENSE). Use these templates anywhere, including commercial deployments. Attribution appreciated but not required.
