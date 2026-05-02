# StudioMeyer Ecosystem

`n8n-templates` is part of the StudioMeyer open source toolkit. Here is everything we build and maintain in the same family.

## MCP Server Products (Hosted)

| Product | Tools | What it does | Link |
|---------|-------|-------------|------|
| **StudioMeyer Memory** | 56 | Persistent AI memory with knowledge graph, semantic search, multi-agent support, 3D visualizations | [memory.studiomeyer.io](https://memory.studiomeyer.io) |
| **StudioMeyer CRM** | 33 | Headless CRM (contacts, companies, deals, pipeline, health scores, Stripe sync) | [crm.studiomeyer.io](https://crm.studiomeyer.io) |
| **StudioMeyer GEO** | 24 | AI visibility monitoring across 8 LLM platforms (ChatGPT, Gemini, Perplexity, Claude, Grok, DeepSeek, Meta AI, Copilot) | [geo.studiomeyer.io](https://geo.studiomeyer.io) |
| **MCP Crew** | 10 | 13 expert personas (CEO, CFO, CMO, CTO, PM, Analyst, Support, Creative, plus 5 Pro tiers) with domain frameworks | [crew.studiomeyer.io](https://crew.studiomeyer.io) |

All MCP products use OAuth 2.1 + Magic Link authentication. Free tiers available. EU Frankfurt hosting.

Connect any of them in Claude Desktop, Claude Code, Cursor, or any MCP client:

```
https://memory.studiomeyer.io/mcp
https://crm.studiomeyer.io/mcp
https://geo.studiomeyer.io/mcp
https://crew.studiomeyer.io/mcp
```

## n8n Integration

| Project | Description | Install |
|---------|-------------|---------|
| **[n8n-templates](https://github.com/studiomeyer-io/n8n-templates)** *(this repo)* | Memory-backed workflow templates that turn StudioMeyer Memory into the long-term memory layer for voice agents, support bots, personal assistants, and more. | clone + import |
| **[n8n-workflows](https://github.com/studiomeyer-io/n8n-workflows)** | Memory-free production workflow templates (Form to CRM, Stripe to Slack, Uptime Monitor, SSL Watcher, Slack Digest). Same four production patterns as this repo, no Memory dependency. | clone + import |
| **[n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory)** | Official n8n community node for StudioMeyer Memory. 16 operations across Memory, Entity, Session, Insight resources. | `npm install n8n-nodes-studiomeyer-memory` |

## Open Source Tools

| Project | Description | Install |
|---------|-------------|---------|
| **[AI Shield](https://github.com/studiomeyer-io/ai-shield)** | LLM security: prompt injection, PII, cost tracking, tool policies, audit logging | `npm install ai-shield-core` |
| **[Darwin Agents](https://github.com/studiomeyer-io/darwin-agents)** | Self-evolving AI agents with A/B testing and safety gates | `npm install darwin-agents` |
| **[Agent Fleet](https://github.com/studiomeyer-io/agent-fleet)** | Multi-agent orchestration for Claude Code CLI | clone + `npm install` |
| **[MCP Personal Suite](https://github.com/studiomeyer-io/mcp-personal-suite)** | 49 personal-productivity tools across mail, calendar, files, tasks, and notes | `npx mcp-personal-suite` |
| **[MCP Video](https://github.com/studiomeyer-io/mcp-video)** | Cinema-grade video production via MCP (FFmpeg + Playwright) | `npx mcp-video` |
| **[Local Memory MCP](https://github.com/studiomeyer-io/local-memory-mcp)** | Self-hosted SQLite-backed Memory for builders who want zero-cloud | `npm install local-memory-mcp` |

## Claude Code Plugin Marketplace

Install all four MCP products plus the n8n custom node as Claude Code plugins with one command:

```bash
/plugin marketplace add studiomeyer-io/studiomeyer-marketplace
```

## Where the templates fit

The flow we expect a builder to take:

1. They find a Reddit post, dev.to article, or LinkedIn carousel about one of the templates here.
2. They clone this repo, import the workflow.json into their n8n instance.
3. They install the [n8n-nodes-studiomeyer-memory](https://github.com/studiomeyer-io/n8n-nodes-studiomeyer-memory) community node.
4. They sign up at [memory.studiomeyer.io](https://memory.studiomeyer.io) for an API key (free tier covers ~10k operations / month).
5. They activate the workflow, point their voice provider / chat platform at the webhook, ship.

Each step adds value. Each step is independently reversible. The templates are MIT, the custom node is MIT, the Memory backend is hosted but the protocol is open (MCP).

## License

Every project in this ecosystem ships under [MIT](LICENSE) unless explicitly stated otherwise. Use them in commercial deployments without permission. Attribution appreciated but not required.

## Contact

- General: [hello@studiomeyer.io](mailto:hello@studiomeyer.io)
- Studio: [studiomeyer.io](https://studiomeyer.io)
- Built in Mallorca.
