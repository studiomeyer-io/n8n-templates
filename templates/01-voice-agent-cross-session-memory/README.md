<!-- studiomeyer-mcp-stack-banner:start -->
> **Part of the [StudioMeyer MCP Stack](https://studiomeyer.io)** · Built in Mallorca · ⭐ if you use it
<!-- studiomeyer-mcp-stack-banner:end -->

# Voice Agent with Cross-Session Memory (Multi-Provider)

> Recognize callers across calls. Reference what they said yesterday. Stop asking returning customers their email three times. **Pick your LLM** (OpenAI default, Anthropic alternative, extensible to any provider).

![Cover](./cover.png)

## What this does

A voice provider (Vapi, Retell, Bland, or any custom telephony bridge that posts JSON) sends a webhook when a call ends. The workflow looks up the caller's phone in StudioMeyer Memory, retrieves prior interactions, builds a context-aware prompt, and replies through the LLM **you choose** (default: OpenAI gpt-5.4-mini, alternative: Anthropic Claude Haiku 4.5). After the reply is sent, it persists the new observation back to memory.

The result is a voice agent that knows your customer the second time they call. No vector-database setup, no Postgres extension, no manual schema work, just one credential and three minutes to import.

## Architecture

```
[Vapi/Retell Webhook]             ← Raw Body enabled for HMAC verification
        │
        ▼
[Verify Webhook (opt-in)]         ← VAPI_SIGNING_SECRET / RETELL_SIGNING_SECRET
        │
        ▼
[Rate Limit (opt-in)]             ← RATE_LIMIT_ENABLED=1
        │
        ▼
[Idempotency Check (opt-in)]      ← IDEMPOTENCY_ENABLED=1, dedup on callId
        │
        ▼
[Normalize Payload]               ← parses E.164 phone, transcript, call ID
        │
        ▼
   ┌──┤ Has Phone? ├──┐
   │                  │
   ▼ true            false ▼
[Memory: Lookup Caller]   (skip memory, treat as anonymous)
        │                       │
   ┌──┤ Known Caller? ├──┐      │
   ▼ yes               no ▼     │
[Memory: Recent Context] [Memory: Create Caller]
   │                     │      │
   └────────┬────────────┴──────┘
            ▼
[Build LLM Prompt]                ← anonymous-aware: switches system prompt
            │
            ▼
[Set Provider] → [Route by Provider]
            ┌────────┴─────────┬─────────┐
            ▼                  ▼         ▼ fallback (typo)
   [OpenAI Reply]      [Anthropic Reply]      │
   gpt-5.4-mini default  claude-haiku-4-5       │
   onError: continue   onError: continue      │
   │       │           │       │              │
 success error       success error            │
   │       │           │       │              │
   ▼       └───────────┘       └──────────────┤
[Normalize LLM Output]                  [LLM Fallback Reply]
   │                                          │
   ▼                                          ▼
[Respond to Voice Provider] ◄─────────────────┤
   plus async writes:                         │
   ├─ Has Phone? (post) ─true→ Memory: Observe + Memory: Learn Outcome
   └─ (anonymous calls skip memory writes)    │
                                              │
                                       Memory: Learn Error
                                       (category: mistake)
```

The error branch is always wired. The `Has Phone?` IF is also always wired so anonymous calls never pollute Memory with `null:caller` entities or queries. The opt-in guards (Verify Webhook, Rate Limit, Idempotency) pass through when their env vars are unset, so the default import boots clean.

The two memory-write nodes (`Memory: Observe` + `Memory: Learn Outcome`) sit on a parallel branch downstream of `Normalize LLM Output`. n8n's `executionOrder: v1` runs each branch depth-first (one branch completes before the next starts) and orders branches by canvas position (top-to-bottom, then left-to-right). `Respond to Voice Provider` is positioned ahead of the memory-write branch on the canvas, so it executes first and the voice provider receives the reply before the memory writes start. This is execution-order-dependent, not a hard async guarantee. For a stricter "respond first, persist later" contract use a separate Execute-Workflow trigger or a queue (Redis, BullMQ, n8n Queue Mode).

## Memory model

| Concept | Storage | Key |
|---|---|---|
| Caller identity | `Entity` of `entityType: caller` | normalized phone (E.164) |
| Per-call detail | `Observation` on the caller entity | timestamp + transcript snippet + agent reply |
| High-level outcome | `Learning` (category: insight) | tagged with `voice-agent` + `caller-<phone>` |

This three-layer model means you can later run `entity.open` to get a full caller dossier, `memory.search` with `recencyWeight: 0.5` to pull recent decisions, or `insight.synthesize` to generate a weekly summary across all callers.

## Setup

> **Self-host gotcha (read first):** the Verify Webhook Code node calls `require('crypto')` for HMAC-SHA256. Self-hosted n8n restricts Node.js builtin imports in Code nodes by default, so you MUST set `NODE_FUNCTION_ALLOW_BUILTIN=crypto` in your n8n environment. Without it, every signed webhook fails with `Cannot find module 'crypto'`. n8n Cloud's allowlist is not publicly documented for individual builtins; verify in your instance with a one-line `require('crypto')` test before enabling HMAC in production. Reference: [n8n Docs - Modules in Code node](https://docs.n8n.io/hosting/configuration/configuration-examples/modules-in-code-node/). See [PRODUCTION_CHECKLIST.md](../../PRODUCTION_CHECKLIST.md) for the full env-var list.

1. **Install the StudioMeyer Memory community node** in your n8n instance:
   ```bash
   # Self-hosted
   npm install n8n-nodes-studiomeyer-memory
   # n8n Cloud / hosted: Settings → Community Nodes → Install package: n8n-nodes-studiomeyer-memory
   ```

2. **Get an API key** at [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Create a credential in n8n: *Credentials → New → StudioMeyer Memory API → Auth Mode: API Key*.

3. **Import the workflow.** In n8n: top-right menu → *Import from clipboard* → paste `workflow.json` → *Import*.

4. **Add an LLM credential.** Default is OpenAI (Credentials → New → OpenAI API). If you'd rather use Claude, add an Anthropic credential instead and change the `Set Provider` node value from `openai` to `anthropic`. You can have both credentials installed and switch between them by editing one node.

5. **Activate the workflow.** The webhook node now exposes a Production URL of the form `https://your-n8n.example.com/webhook/vapi-callback`.

6. **Wire the webhook into your voice provider.**
   - **Vapi:** Dashboard → Assistants → your assistant → Server URL → paste the production URL. Set *Server URL Events* to include `end-of-call-report`.
   - **Retell:** Dashboard → Agents → your agent → Webhook URL → paste. Set events to `call_ended`.
   - **Custom telephony:** POST a JSON payload with `caller_phone` and `transcript` fields. The Code node will normalize either Vapi or Retell shapes; for a third format, edit the two `body?...` extractors at the top.

7. **Test.** Call your voice number. End the call. The first call creates the caller entity. The second call retrieves it and the system prompt now contains the prior interaction summary. Verify in [memory.studiomeyer.io](https://memory.studiomeyer.io) → *Knowledge Graph* → search for the caller's phone number.

## Multi-provider switch

The workflow has a `Set Provider` node followed by a `Route by Provider` switch. Default value is `openai`. Change to `anthropic` (or add your own branch with a third LLM) without rebuilding the rest of the flow. Both branches converge in `Normalize LLM Output` which extracts the reply text from either provider's response shape and feeds it to the response + memory writes.

**Switch from OpenAI to Anthropic:**

1. Open the `Set Provider` node.
2. Change the `provider` field from `openai` to `anthropic`.
3. Save and re-test. The workflow now routes through `Anthropic Reply` and Claude answers the call.

**Add a third provider (Gemini, Mistral, local Ollama):**

1. Add a new Reply node using the appropriate credential type.
2. Add a new rule in `Route by Provider` that matches `provider == "gemini"` and routes to it.
3. On the new Reply node, set `On Error: Continue (Error Output)`. Connect main output 0 to `Normalize LLM Output` and main output 1 to `LLM Fallback Reply`.
4. Update `Normalize LLM Output` to handle the new provider's response shape (the existing code already handles OpenAI `choices[0].message.content` and Anthropic `content[0].text`, just append a third branch).
5. Set `provider` to `gemini` in `Set Provider` to test.

The error branch and Memory writes stay identical, you only add nodes, never edit the convergence point.

## Extending

**Add caller scoring.** After `Memory: Lookup Caller`, branch on observation count. Callers with 5+ prior interactions get a different greeting ("Welcome back, this is your sixth call!"). Use `entity.open` to fetch the full observation list.

**Hand off to a human.** Add an IF node after Claude that checks for sentiment keywords ("frustrated", "speak to a manager"). Branch into a Slack notification with the caller's history pre-formatted, so the human takes over with full context.

**Multi-language replies.** Add a language-detection step before `Build LLM Prompt`. Store the detected language as an observation on the caller entity. Subsequent calls use the stored preference automatically.

**Daily caller-list synthesis.** Add a separate scheduled workflow that runs `Memory: Synthesize` every morning with `query: "voice-agent recent calls"` and posts the cluster summary to your team Slack. The voice agent's data becomes a CRM-lite without any extra plumbing.

## Cost notes

Per call (assuming ~30s of conversation, ~500 input + 200 output tokens):

| Component | Cost (Stand 2026-05) | Per-call cost |
|---|---|---|
| **StudioMeyer Memory** | EUR 0 / 29 / 49 per month | Free tier: 200 credits (one credit per op, ~50 voice calls). Pro tier: unlimited. |
| **OpenAI gpt-5.4-mini** (default) | $0.75 / 1M input + $4.50 / 1M output | ~$0.0015 per call |
| **Anthropic claude-haiku-4-5** | $1 / 1M input + $5 / 1M output | ~$0.0015 per call |
| Vapi or Retell | provider varies | ~$0.07-0.30 per minute (BYOK stack closer to $0.30) |

**Worked example at 1000 calls/month, 30s avg duration:**

| Stack | Memory | LLM | Voice provider | Total /mo |
|---|---|---|---|---|
| OpenAI gpt-5.4-mini | EUR 0 (free tier) | ~$1.50 | ~$35-150 | **~$36-152/mo** |
| Anthropic claude-haiku-4-5 | EUR 0 | ~$1.50 | ~$35-150 | **~$37-153/mo** |

Voice provider cost dominates at this scale. LLM + Memory together stay below $2/mo. At 5000+ calls/month you cross the Memory free tier and need Pro at EUR 29/mo. Pro also unlocks the 3D caller-relationship graph at memory.studiomeyer.io/portal/memory/knowledge.

The error branch fires on LLM failures (rate limit, 5xx). It writes one extra Memory op (Learn Error) per failure. At a healthy 99.5% success rate this adds <0.5% to your bill.


## Common gotchas

- **No phone in payload.** Some Vapi setups send `caller=anonymous` or strip the number. The new `Has Phone?` IF branches on `hasPhone` and routes anonymous calls directly to the LLM with an "anonymous caller" system prompt, skipping Memory entirely on both the lookup and write paths. This avoids polluting Memory with `null:caller` entities. If you want recognition for anonymous callers, use the Vapi `call.id` as a synthetic identifier in `Normalize Payload`.
- **Transcript is empty for short calls.** Vapi sends `end-of-call-report` even for 2-second calls where nothing was said. The LLM still replies with an empty user message. Either add a guard in `Normalize Payload` to skip when transcript is empty, or accept the noise as a graceful default reply.
- **Multiple calls in flight.** n8n's default execution mode is fire-and-forget per webhook trigger. If the same caller calls twice in 30 seconds, both runs may see "0 entities" on the first lookup (race condition). Memory's gatekeeper deduplicates on the entity-create side, so you don't get duplicate `caller` entities, but the second call's "first call" observation is technically wrong. For high-volume use, enable Idempotency Check via `IDEMPOTENCY_ENABLED=1` (catches retries on the same `callId`) and consider the `entity.observe` with auto-create fallback (Memory v3.17 feature, in private beta).
- **Anthropic node type-string.** The Anthropic Reply node uses `@n8n/n8n-nodes-langchain.anthropic` (the LangChain-vendored direct-API node), not `n8n-nodes-base.anthropic` (which does not exist in n8n core and produces "Unrecognized node type" on activation). The OpenAI counterpart is the core `n8n-nodes-base.openAi` (verified working in n8n 2.15.0). If you fork this template and a Reply branch fails to activate, double-check the type-string against your n8n version.
- **Memory writes are execution-order-dependent, not strictly async.** The two memory-write nodes (`Memory: Observe` + `Memory: Learn Outcome`) live downstream of `Has Phone? (post)`, which itself is on a parallel branch to `Respond to Voice Provider` after `Normalize LLM Output`. n8n's `executionOrder: v1` runs each branch depth-first and orders branches by canvas position (top-to-bottom, left-to-right). `Respond` is positioned ahead of the memory-write branch on the canvas so the voice provider gets the reply first, then the memory writes happen. **If you reposition nodes the order can change.** This is execution-order-dependent, not a hard async guarantee. For a stricter "respond first, persist always-after" contract, route the writes through a separate Execute-Workflow trigger or a queue (Redis, BullMQ, n8n Queue Mode).

## Production patterns

Four patterns ship in `workflow.json` as actual nodes, three opt-in via env vars and one always-on error branch. A fifth pattern, Memory de-duplication, is server-side and needs no workflow node. The opt-in nodes pass through when their env var is unset, so the default import boots clean.

**Idempotency** (opt-in, `IDEMPOTENCY_ENABLED=1`). The `Idempotency Check` Code node holds a 5-minute in-memory window of seen `callId` values (Vapi `message.call.id` or Retell `call.call_id`) and short-circuits duplicates. Voice providers can re-deliver `end-of-call-report` on transient 5xx so this catches double-fires without touching Memory or the LLM. For clustered n8n deployments, swap the `$getWorkflowStaticData` block for Redis `SET NX EX 300`. The node has the swap pattern in its comments.

**Rate limiting** (opt-in, `RATE_LIMIT_ENABLED=1`). The `Rate Limit` Code node caps each caller-phone (or IP if anonymous) at 60 calls in a 5-minute sliding window. For voice this matters less than for chat (callers don't burst-call) but a leaked webhook URL can still spike your LLM bill. The map is bounded at 5 000 entries with eviction. For real production loads, put rate limiting on a reverse proxy (Nginx `limit_req_zone`, Cloudflare WAF, Traefik) and keep this node as defense-in-depth.

**Webhook HMAC verification** (opt-in, `VAPI_SIGNING_SECRET` or `RETELL_SIGNING_SECRET`). The `Verify Webhook` Code node computes HMAC-SHA256 of the raw body using the configured secret and compares against `x-vapi-signature` or `x-retell-signature` with `crypto.timingSafeEqual`. The Webhook node has `rawBody: true` enabled so the byte-stream is preserved. Length-guard before the timing-safe compare prevents `RangeError` DoS from a 1-char signature. Without HMAC, an attacker who guesses your webhook URL can spike your bill.

**Error branches** (always on). Both LLM Reply nodes have `On Error: Continue (Error Output)` enabled. The error pin lands at `LLM Fallback Reply`, which builds a SHORT voice-friendly message ("Sorry, I'm having trouble, please call back in a minute") and feeds two destinations: `Respond to Voice Provider` (so the caller hears something instead of silence) and `Memory: Learn Error` with `category: mistake, tags: [llm-error, <provider>]`. The `Route by Provider` fallback output (typo or unknown provider value) also lands at `LLM Fallback Reply`. The fallback handler discriminates between LLM-error and router-fallback so private memory context never leaks into the audit trail. The error syntax is `{{ $json.error.message }}`, not `{{ $error.message }}` (does not exist) and not `{{ $json.execution.error.message }}` (Error Trigger Workflow only, not inline pins).

**Memory de-duplication** (always on, server-side). StudioMeyer Memory's gatekeeper deduplicates writes on >95% content similarity automatically. If your workflow somehow fires twice on the same observation despite idempotency (clock skew, manual re-run), the second write is silently skipped. You can verify by checking `action: SKIPPED, similarity: 1` in the Memory response. This is server-side, no env var needed.

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
| Vapi or Retell | latest stable | varies | trial available | always |

Free trial credit covers ~30 minutes of calls.

## Credentials checklist

Before activation, create these credentials in n8n:

- [ ] **StudioMeyer Memory API** (`studioMeyerMemoryApi`). Get key at [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Test: any memory.search call returns `success: true`.
- [ ] **OpenAI API** (`openAiApi`) OR **Anthropic API** (`anthropicApi`). Get key at [platform.openai.com](https://platform.openai.com) / [console.anthropic.com](https://console.anthropic.com). Test: model list endpoint returns models.
- [ ] **Vapi / Retell webhook** (provider-specific). After activation, copy the Webhook node's Production URL into your voice provider dashboard.
- [ ] **Webhook signing secret (recommended).** Set the n8n env var `VAPI_SIGNING_SECRET` (or `RETELL_SIGNING_SECRET`) to a strong random string and configure the same secret in your voice provider dashboard. The Verify Webhook Code node then validates the HMAC-SHA256 signature against the raw body on every call. Without HMAC, an attacker who guesses your webhook URL can spike your bill.

## Live verification

Template 01 was structurally validated against n8n 2.15.0 in [n8n.studiomeyer.io](https://n8n.studiomeyer.io) during the v0.4.0-prep pass. Pre-activation check passed for all 29 nodes including the `@n8n/n8n-nodes-langchain.anthropic` type-string for the Anthropic Reply node and the `n8n-nodes-base.openAi` type-string for OpenAI. The Memory pattern (entity.search + create + observe + memory.learn) was smoke-tested in Template 02 against the same backend at memory.studiomeyer.io with executions 445 + 446 both green. End-to-end voice test (Vapi or Retell webhook → LLM reply → Memory write) requires a voice-provider account and is part of the next pass when the trigger account is provisioned.

## How this compares

Memory layers a 2026 builder considers for an n8n voice agent:

| Feature | StudioMeyer Memory | Mem0 | Zep | Memori |
|---|---|---|---|---|
| Verified n8n custom node | Yes (this repo, npm provenance) | Community HTTP node | Community node | Third-party node |
| Reference templates ship | 8 templates in this repo | Reddit posts only (Mem0+Vapi+n8n got 6 upvotes) | Some | None curated |
| Free tier | 200 credits (one per op) | 10K memories + 1K retrieval calls/mo | 1k credits/mo cloud + Graphiti OSS self-host | OSS self-host only |
| Bi-temporal `asOf` queries | Yes | Limited | Yes (via Graphiti) | No |
| Knowledge graph (entities + relations) | Native | Hybrid vector + graph | Native (Graphiti) | Vector only |
| E.164-normalized caller key | Yes (`Entity` of `entityType: caller`) | Manual | Manual | Manual |
| Multi-tenant isolated by default | Yes | Manual config | Manual config | Self-host |
| EU hosting | Yes (Frankfurt, Hetzner) | US-default | US-default | Self-host |
| OAuth 2.1 + API key | Both | API key | API key | API key |

The Mem0+Vapi+n8n Reddit post with 6 upvotes is the existing benchmark for this niche. This template is the StudioMeyer answer with a verified custom node, an entity-typed caller pattern, an EU-hosted memory backend, and four opt-in production patterns ready to toggle. Voice agents are listed in Mem0's State of AI Agent Memory 2026 as "the fastest growing integration category", which is why this template ships first in the StudioMeyer reference set.

All four projects are open about their tradeoffs. The fastest comparison: wire each one to a throwaway voice number for a day and see which response shape feels right.

## Related templates

- [02 - AI Customer Support with Customer History](../02-customer-support-with-history/) · same memory pattern for chat platforms (Telegram, WhatsApp, Slack)
- [03 - Personal Assistant with Long-Term Memory](../03-personal-assistant-long-term-memory/) · single-user variant with intent classifier and tool use

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Part of the [StudioMeyer MCP Stack](../../README.md). Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues + ideas at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues).*
