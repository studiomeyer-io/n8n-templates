<!-- studiomeyer-mcp-stack-banner:start -->
> **Part of the [StudioMeyer MCP Stack](https://studiomeyer.io)**, Built in Mallorca · ⭐ if you use it
<!-- studiomeyer-mcp-stack-banner:end -->

# Template Title (Multi-Provider)

> One-sentence value prop. What does the builder get? Be concrete.

![Cover](./cover.png)

## What this does

Two short paragraphs. First paragraph = the data flow as a single sentence with arrows ("trigger fires → workflow looks up Memory → builds context-aware prompt → LLM replies → outcome persisted"). Second paragraph = the killer phrase someone searching for this template will resonate with ("the result is X for the second time they Y").

## Architecture

```
[Trigger]                         ← raw body / signed request if applicable
    │
    ▼
[Verify Webhook (opt-in)]         ← <PROVIDER>_SIGNING_SECRET
    │
    ▼
[Rate Limit (opt-in)]             ← RATE_LIMIT_ENABLED=1, 60/5min/<key>
    │
    ▼
[Idempotency Check (opt-in)]      ← IDEMPOTENCY_ENABLED=1, dedup on <id>
    │
    ▼
[Normalize Payload]
    │
    ▼
[Memory: Lookup]
    │
    ▼
   ┌──┤ Decision ├──┐
   ▼ yes          no ▼
[Path A]        [Path B]
   │               │
   └──────┬────────┘
          ▼
[Build LLM Prompt]
          │
          ▼
[Set Provider] → [Route by Provider]
          ┌────────┴─────────┬─────────┐
          ▼                  ▼         ▼ fallback
   [OpenAI Reply]    [Anthropic Reply]      │
   onError: continue   onError: continue    │
   │       │           │       │            │
   ▼       └───────────┘       └────────────┤
[Normalize LLM Output]              [LLM Fallback Reply]
   │                                        │
   ▼                                        ▼
[Reply + Memory writes] ◄───────────────────┤
                                            │
                                     Memory: Learn Error
                                     (category: mistake)
```

## Memory model

| Concept | Storage | Key |
|---|---|---|
| <Identity> | `Entity` of `entityType: <type>` | <key> |
| <Per-event detail> | `Observation` on the entity | <observation key> |
| <High-level outcome> | `Learning` (category: insight) | <tags> |

## Setup

1. **Install the StudioMeyer Memory community node.** `npm install n8n-nodes-studiomeyer-memory` for self-hosted, or *Settings → Community Nodes → Install* on n8n Cloud.
2. **Get an API key** at [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Add as credential `StudioMeyer Memory API`.
3. **Add an LLM credential.** OpenAI by default. Anthropic if you'd rather use Claude (toggle in `Set Provider`).
4. **Import this workflow** (workflow.json in this folder).
5. **Configure the SET-ME markers.** Yellow sticky notes flag every spot that needs your config.
6. **Wire up your trigger** (provider-specific instructions below).
7. **Test.** First execution creates the entity, second references it.

### Provider-specific webhook setup

- **<Provider A>:** Dashboard → Settings → Webhook URL → paste the production URL of your Trigger node. Set events: `<event-name>`.
- **<Provider B>:** Similar but with these specifics: `<...>`.

## Multi-provider switch

The workflow has a `Set Provider` node followed by a `Route by Provider` switch. Default value is `openai`. Change to `anthropic` (or add your own branch with a third LLM) without rebuilding the rest of the flow. Both branches converge in `Normalize LLM Output` which extracts the reply text from either provider's response shape.

**Switch from OpenAI to Anthropic:**

1. Open the `Set Provider` node.
2. Change the `provider` field from `openai` to `anthropic`.
3. Save and re-test.

**Add a third provider (Gemini, Mistral, local Ollama):**

1. Add a new Reply node using the appropriate credential type.
2. Add a new rule in `Route by Provider` that matches `provider == "gemini"` and routes to it.
3. On the new Reply node, set `On Error: Continue (Error Output)`. Connect main output 0 to `Normalize LLM Output` and main output 1 to `LLM Fallback Reply`.
4. Update `Normalize LLM Output` to handle the new provider's response shape.
5. Set `provider` to `gemini` in `Set Provider` to test.

## Extending

**<First extension idea>.** One paragraph in flowing prose explaining what to add and where. Reference specific node names from the workflow.

**<Second extension idea>.** Same shape.

**<Third extension idea>.** Same shape.

**<Fourth extension idea>.** Same shape.

## Cost notes

Per execution (assuming average payload):

| Component | Cost (Stand 2026-05) | Per-execution cost |
|---|---|---|
| **StudioMeyer Memory** | EUR 0 / 29 / 49 per month | Free tier: 200 credits (one credit per op). Pro tier: unlimited. |
| **OpenAI gpt-5.4-mini** (default) | $0.75 / 1M input + $4.50 / 1M output | ~$<X> per execution |
| **Anthropic claude-haiku-4-5** | $1 / 1M input + $5 / 1M output | ~$<X> per execution |

**Worked example at <typical-volume>/month:**

| Stack | Memory | LLM | Total /mo |
|---|---|---|---|
| OpenAI gpt-5.4-mini | EUR 0 (free tier) | ~$<X>/mo | **~$<X>/mo** |
| Anthropic claude-haiku-4-5 | EUR 0 | ~$<X>/mo | **~$<X>/mo** |

The error branch fires on LLM failures (rate limit, 5xx). It writes one extra Memory op (Learn Error) per failure. At a healthy 99.5% success rate this adds <0.5% to your bill.

## Common gotchas

- **<First gotcha>.** Explain what goes wrong, why, and how to fix it. One paragraph.
- **<Second gotcha>.** Same.
- **<Third gotcha>.** Same.
- **<Fourth gotcha>.** Same.
- **Anthropic node type-string.** The Anthropic Reply node uses `@n8n/n8n-nodes-langchain.anthropic` (the LangChain-vendored direct-API node), not `n8n-nodes-base.anthropic` (which does not exist in n8n core). The OpenAI counterpart is the core `n8n-nodes-base.openAi` (verified working in n8n 2.15.0).

## Production patterns

Four patterns ship in `workflow.json` as actual nodes, three opt-in via env vars and one always-on error branch. The opt-in nodes pass through when their env var is unset, so the default import boots clean.

**Idempotency** (opt-in, `IDEMPOTENCY_ENABLED=1`). The `Idempotency Check` Code node holds a 5-minute in-memory window of seen `<idempotency-key>` values and short-circuits duplicates. For clustered n8n deployments, swap the `$getWorkflowStaticData` block for Redis `SET NX EX 300`. The node has the swap pattern in its comments.

**Rate limiting** (opt-in, `RATE_LIMIT_ENABLED=1`). The `Rate Limit` Code node caps each `<rate-limit-key>` at 60 requests in a 5-minute sliding window. The map is bounded at 5 000 entries with eviction. For real production loads, put rate limiting on a reverse proxy (Nginx `limit_req_zone`, Cloudflare WAF, Traefik) and keep this node as defense-in-depth.

**Webhook HMAC verification** (opt-in, `<PROVIDER>_SIGNING_SECRET`). The `Verify Webhook` Code node computes HMAC-SHA256 of the raw body using the configured secret and compares against the provider signature header with `crypto.timingSafeEqual`. Length-guard before the timing-safe compare prevents `RangeError` DoS from a 1-char signature. Without HMAC, an attacker who guesses your webhook URL can spike your bill.

**Error branches** (always on). Both LLM Reply nodes have `On Error: Continue (Error Output)` enabled. The error pin lands at `LLM Fallback Reply`, which builds a graceful customer-facing message and feeds two destinations: the reply node (so the user gets an answer) and `Memory: Learn Error` with `category: mistake, tags: [llm-error, <provider>]`. The fallback handler discriminates between LLM-error and router-fallback so private memory context never leaks into the audit trail. The error syntax is `{{ $json.error.message }}`, not `{{ $error.message }}` (does not exist) and not `{{ $json.execution.error.message }}` (Error Trigger Workflow only, not inline pins).

**Memory de-duplication** (always on, server-side). StudioMeyer Memory's gatekeeper deduplicates writes on >95% content similarity automatically. Server-side, no env var needed.

## Hard compatibility floor

**Minimum n8n version with CVE-2026-27493 fix:** >= 2.9.3 (stable channel) / >= 2.10.1 (latest / beta channel) / >= 1.123.22 (1.x LTS). CVE-2026-27493 is an unauthenticated RCE in Form nodes (CVSS 9.5). This template does not use Form nodes itself, but you should still upgrade for general security. The pre-activation check on n8n 2.15.0 was used to validate every node type-string in this template.

## Tech stack matrix

| Component | Version | Cost | Free tier | Required when |
|---|---|---|---|---|
| n8n | >= 2.10.1 (CVE-2026-27493 floor) | self-hosted free / Cloud $20/mo | n8n Cloud trial | always |
| n8n-nodes-studiomeyer-memory | >= 0.1.0 | free | n/a | always |
| StudioMeyer Memory | API key | EUR 0 / 29 / 49 | 200 credits (one per op) | always |
| OpenAI (default) | gpt-5.4-mini | $0.75 / 1M input + $4.50 / 1M output | $5 trial credit | provider = openai |
| Anthropic | claude-haiku-4-5 | $1 / 1M input + $5 / 1M output | $5 trial credit | provider = anthropic |
| <External provider> | latest stable | varies | trial available | always |

## Credentials checklist

Before activation, create these credentials in n8n:

- [ ] **StudioMeyer Memory API** (`studioMeyerMemoryApi`). Get key at [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Test: any memory.search call returns `success: true`.
- [ ] **OpenAI API** (`openAiApi`) OR **Anthropic API** (`anthropicApi`). Get key at [platform.openai.com](https://platform.openai.com) / [console.anthropic.com](https://console.anthropic.com). Test: model list endpoint returns models.
- [ ] **<External provider>** (provider-specific). Setup webhook URL in their dashboard pointing at your n8n instance.
- [ ] **Webhook signing secret (recommended).** Set the n8n env var `<PROVIDER>_SIGNING_SECRET` to a strong random string and configure the same secret in your provider dashboard. The Verify Webhook Code node then validates the HMAC-SHA256 signature against the raw body on every call.

## Related templates

- [01 - Voice Agent Cross-Session Memory](../01-voice-agent-cross-session-memory/) · voice provider variant (Vapi / Retell)
- [02 - AI Customer Support with History](../02-customer-support-with-history/) · multi-customer chat variant
- [03 - Personal Assistant with Long-Term Memory](../03-personal-assistant-long-term-memory/) · single-user variant with intent classifier and tool use

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Part of the [StudioMeyer MCP Stack](../../README.md). Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues + ideas at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues).*
