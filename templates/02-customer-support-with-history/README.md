<!-- studiomeyer-mcp-stack-banner:start -->
> **Part of the [StudioMeyer MCP Stack](https://studiomeyer.io)** · Built in Mallorca · ⭐ if you use it
<!-- studiomeyer-mcp-stack-banner:end -->

# AI Customer Support with Customer History (Multi-Provider)

> Returning customers don't have to start from scratch. The bot greets them by name, references their last ticket, and your human agents take over with the full file. **Pick your LLM** (OpenAI default, Anthropic alternative).

![Cover](./cover.png)

## What this does

A customer messages your Telegram bot (or WhatsApp Cloud API, or Intercom, the trigger is swappable). The workflow extracts a stable customer key from the message (email if visible, otherwise the Telegram user ID), looks up that customer in StudioMeyer Memory, retrieves their full dossier, and asks Claude to reply with the dossier as system-prompt context.

After the reply is sent, two async writes persist the new ticket as an observation on the customer entity and a high-level learning that's searchable across your support corpus.

The bot now has institutional memory. The third time a customer writes "my login isn't working" you can spot the pattern automatically.

## Architecture

```
[Telegram Trigger]                ← secretToken option = built-in HMAC
        │
        ▼
[Verify Webhook (opt-in)]         ← WEBHOOK_INTEGRITY_CHECK_ENABLED=1
        │
        ▼
[Rate Limit (opt-in)]             ← RATE_LIMIT_ENABLED=1, 60/5min/chat
        │
        ▼
[Idempotency Check (opt-in)]      ← IDEMPOTENCY_ENABLED=1, dedup on update_id
        │
        ▼
[Extract Customer Key]            ← email regex, fallback to Telegram user ID
        │
        ▼
[Memory: Lookup Customer]         ← entity.search, entityType=customer
        │
        ▼
   ┌──┤ Known Customer? ├──┐
   │                       │
   ▼ yes                  no ▼
[Memory: Customer Dossier]   [Memory: Create Customer]
   (entity.open, full file)   (with first observation)
   │                       │
   └────────┬──────────────┘
            ▼
[Build LLM Prompt]                ← injects dossier into system prompt
            │
            ▼
[Set Provider] → [Route by Provider]
            ┌────────┴─────────┐
            ▼                  ▼
   [OpenAI Reply]      [Anthropic Reply]
   gpt-5-mini default  claude-haiku-4-5
   onError: continue   onError: continue
   │       │           │       │
 success error       success error
   │       │           │       │
   ▼       ▼           ▼       ▼
[Normalize] ◄──────────┘   [LLM Fallback Reply]
   │                              │
   ├─ Telegram Reply ◄─────────────┤
   ├─ Memory: Observe Ticket       │
   └─ Memory: Learn Ticket         └─ Memory: Learn Error
                                      (category: mistake)
```

The error branch is always wired. If OpenAI rate-limits or Anthropic returns 5xx, the customer still gets a graceful reply and the failure lands in your knowledge graph as a `mistake` learning so you can spot patterns.

## Memory model

| Concept | Storage | Key |
|---|---|---|
| Customer identity | `Entity` of `entityType: customer` | email (if extractable) else `tg:<user_id>` |
| Per-ticket detail | `Observation` on the customer entity | timestamp + question snippet + bot reply snippet |
| Corpus learnings | `Learning` (category: insight) | tagged with `support` + `customer-<key>` |

The dual write (observation + learning) costs you one extra memory op per ticket but unlocks two very different lookups:

- **Per-customer view** (`entity.open`): "Show me everything about acme@example.com." Returns the full dossier in one call.
- **Corpus view** (`memory.search`): "What login issues did we see this week?" Searches across all customers, ranked by recency + relevance.

## Setup

1. **Install the StudioMeyer Memory community node** (see repo README).

2. **Create a Telegram bot.**
   - Message @BotFather on Telegram, type `/newbot`, follow the prompts.
   - Save the token (looks like `123456789:ABC-DEF...`).

3. **Add credentials in n8n:**
   - StudioMeyer Memory API → API Key from [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys).
   - Telegram → paste the bot token.
   - OpenAI API (default provider) → key from [platform.openai.com](https://platform.openai.com), `gpt-5-mini` is the current default mini tier.
   - Anthropic API (alternative provider) → key from [console.anthropic.com](https://console.anthropic.com), `claude-haiku-4-5` is the current Haiku tier (faster and cheaper than Sonnet for support replies).

4. **Import the workflow** (`workflow.json` in this folder).

5. **Activate** the workflow. Telegram automatically registers the webhook.

6. **Test** by messaging your bot. The first message creates a customer entity. Subsequent messages show the dossier in the LLM prompt, verify by checking the *Build LLM Prompt* node's output after a second message.

## Swap Telegram for WhatsApp / Intercom / web chat

Replace two nodes; the middle 8 stay identical:

| Provider | Trigger node | Reply node |
|---|---|---|
| **WhatsApp Cloud API** | WhatsApp Trigger (built-in) | WhatsApp Send Message |
| **Intercom** | Webhook (manual), Intercom posts to your URL | HTTP Request to Intercom Conversations API |
| **Custom web chat** | Webhook | Respond to Webhook |
| **Slack** | Slack Trigger | Slack Send Message |

In each case, update the `Extract Customer Key` Code node to read the right field for customer identity (e.g. WhatsApp `from`, Intercom `user.email`, Slack `user.id`).

## Multi-provider switch

The workflow ships with two providers wired in parallel: OpenAI `gpt-5-mini` (default) and Anthropic `claude-haiku-4-5`. Pick one or run both behind a feature flag.

**Switch from OpenAI to Anthropic:**

1. Open the `Set Provider` node.
2. Change the `provider` field from `openai` to `anthropic`.
3. Save and re-test, the workflow now routes through `Anthropic Reply`.

**Add a third provider (Gemini, Mistral, local Ollama):**

1. Add a new Reply node (e.g. `Gemini Reply`) using whichever credential type fits.
2. Add a new rule in `Route by Provider` that matches `provider == "gemini"` and routes to the new node.
3. On the new Reply node, set `On Error: Continue (Error Output)`. Connect main output 0 to `Normalize LLM Output` and main output 1 to `LLM Fallback Reply`.
4. Update `Normalize LLM Output` to handle the new provider's response shape (the existing code already handles OpenAI `choices[0].message.content` and Anthropic `content[0].text`, just append a third branch for the new shape).
5. Set `provider` to `gemini` in `Set Provider` to test.

The error branch and Memory writes stay identical, you only add nodes, never edit the convergence point.

## Extending

**Hand off to humans on demand.** Add a sentiment-check IF after Claude. If the reply suggests escalation ("I'd recommend speaking with a manager"), post the customer's full dossier into a #support Slack channel with a button that mutes the bot for that customer for 24h.

**Pattern detection.** Add a weekly scheduled workflow that runs `Memory: Synthesize` with `query: "support recent issues"`. The synthesis clusters tickets by topic and posts the top 5 patterns to your team's morning brief, you'll spot recurring product issues that single tickets hide.

**Pre-fill CRM contact.** When a new customer entity is created, fan out to the [StudioMeyer CRM](https://crm.studiomeyer.io) MCP server and create a contact with the email + first message. Now your sales team gets a contact record before the bot even finishes replying.

**Multi-language.** Add a language-detection step before `Build LLM Prompt`. Store the detected language as an observation. The system prompt then instructs Claude to reply in the customer's language.

**Customer health score.** After `Memory: Customer Dossier`, run a small Code node that scores each customer on three signals: ticket frequency (observations per week), last interaction recency, and sentiment trend (parse a sentiment tag from the most recent N observations). Persist the score as a new observation `health: 4/10`. Then add an IF after Build LLM Prompt: customers below 5 get a different system prompt that prioritizes empathy over speed and offers escalation up-front. Pre-empt churn before it happens.

## Cost notes

Per ticket the workflow does 4 Memory ops (Lookup + Open or Create + Observe + Learn) plus 1 LLM call. Cost depends on which provider you pick.

| Component | Cost (Stand 2026-04) | Per-ticket cost |
|---|---|---|
| **StudioMeyer Memory** | EUR 0 / 29 / 49 per month | Free tier: 200 credits (one credit per op, ~50 support tickets). Pro tier: unlimited. |
| **OpenAI gpt-5-mini** (default) | $0.25 / 1M input + $2.00 / 1M output | ~$0.0014 per ticket (~500 in + 600 out tokens) |
| **Anthropic claude-haiku-4-5** | $1 / 1M input + $5 / 1M output | ~$0.0035 per ticket |
| Telegram Bot API | free | free |

**Worked example at 5 000 tickets/month:**

| Stack | Memory | LLM | Total |
|---|---|---|---|
| OpenAI gpt-5-mini | EUR 29/mo (Pro tier required) | ~$7/mo | **~EUR 36/mo** |
| Anthropic claude-haiku-4-5 | EUR 29/mo | ~$17.50/mo | **~EUR 47/mo** |

Below 50 tickets/month you stay within the free Memory tier (200 credits, one per op) and pay only the LLM (~$1-9/mo). Past that, Pro at EUR 29/mo lifts the cap. Pro tier ($29/mo) also unlocks the 3D customer-relationship graph at memory.studiomeyer.io/portal/memory/knowledge.

The error branch fires on LLM failures (rate limit, 5xx). It writes one extra Memory op (Learn Error) per failure. At a healthy 99.5% success rate this adds <0.5% to your bill.


## Common gotchas

- **No email and no `from.id` in message.** Channel posts and forwarded messages without a sender have no Telegram user id. The customer-key extractor falls back to `chat:<chatId>` so two anonymous senders in the same chat collapse into one entity, which is the right behavior for a channel-as-customer setup. If you want strict per-sender separation, add an early IF that drops messages without `message.from.id` instead of falling back. The extractor throws when even chat id is missing.
- **Customer with two Telegram accounts looks like two people.** When an email IS provided in a later message, run a separate maintenance workflow that uses `entity.relate` to merge the `tg:<id>` entity into the email entity. Then re-tag downstream observations.
- **Markdown rendering.** Telegram parses Markdown but only a subset (no tables, limited links). The `parse_mode: Markdown` field works for bold/italic, if you need richer formatting switch to `MarkdownV2` and escape special characters in the reply text.
- **Bot rate limit.** Telegram bots get throttled at ~30 messages/sec across all chats. For high-volume scenarios, use the WhatsApp Cloud API or batch replies.
- **Duplicate observations on retry.** If n8n retries the workflow, the entity-observe step might write the same ticket twice. Memory's gatekeeper deduplicates on >95% similarity, so identical observations get skipped automatically. For belt-and-suspenders, toggle the Idempotency Code node on with `IDEMPOTENCY_ENABLED=1`.
- **Anthropic node type-string.** The Anthropic Reply node uses `@n8n/n8n-nodes-langchain.anthropic` (the LangChain-vendored direct-API node), not `n8n-nodes-base.anthropic` (which does not exist in n8n core and produces "Unrecognized node type" on activation). The OpenAI counterpart in this template is the core `n8n-nodes-base.openAi` (verified working in n8n 2.15.0, recognized by pre-activation check). Newer n8n versions also expose `@n8n/n8n-nodes-langchain.openAi`, both work as of May 2026. If you fork this template and a Reply branch fails to activate, double-check the type-string against your n8n version.

## Production patterns

Four patterns ship in `workflow.json` as actual nodes, three opt-in via env vars and one always-on error branch. A fifth pattern, Memory de-duplication, is server-side and needs no workflow node. The opt-in nodes pass through when their env var is unset, so the default import boots clean.

**Idempotency** (opt-in, `IDEMPOTENCY_ENABLED=1`). The `Idempotency Check` Code node holds a 5-minute in-memory window of seen `update_id` values and short-circuits duplicates. Telegram retries on 5xx so this catches double-fires without touching Memory or the LLM. For clustered n8n deployments, swap the `$getWorkflowStaticData` block for Redis `SET NX EX 300`. The node has the swap pattern in its comments.

**Rate limiting** (opt-in, `RATE_LIMIT_ENABLED=1`). The `Rate Limit` Code node caps each Telegram chat at 60 requests in a 5-minute sliding window. Without this, a misbehaving client or a leaked bot URL spikes your LLM bill. The map is bounded at 5 000 entries and evicts expired buckets when full. For real production loads, put rate limiting on a reverse proxy (Nginx `limit_req_zone`, Cloudflare WAF, Traefik) and keep this node as defense-in-depth.

**Webhook security** (opt-in, configured on the trigger). Telegram's `secretToken` mechanism is built into the Telegram Trigger node. Set `additionalFields.secretToken` to a strong random string and re-register the webhook with Telegram's `setWebhook?secret_token=...`. Telegram then sends `X-Telegram-Bot-Api-Secret-Token` on every call and the trigger validates it automatically. The optional `Verify Webhook` Code node downstream (toggled with `WEBHOOK_INTEGRITY_CHECK_ENABLED=1`) is the second defense layer: it rejects malformed payloads that pass HMAC but lack the fields downstream nodes need. For non-Telegram triggers (WhatsApp, Slack, custom HTTP), replace the Telegram Trigger with a generic Webhook node and use a generic HMAC + length-guard pattern (see Template 01 Verify Webhook Code node for a working example).

**Error branches** (always on). Both LLM Reply nodes have `On Error: Continue (Error Output)` enabled. The error pin lands at `LLM Fallback Reply`, which builds a graceful customer-facing message and feeds two destinations: Telegram Reply (so the customer gets an answer) and Memory: Learn Error with `category: mistake, tags: [llm-error, <provider>]` (so you spot patterns in the knowledge graph). The `Route by Provider` fallback output (when the `provider` field is neither `openai` nor `anthropic`, e.g. typo) also lands at `LLM Fallback Reply`, so a misconfigured provider value still produces a reply instead of silent dead-end. The error syntax is `{{ $json.error.message }}`, not `{{ $error.message }}` (which does not exist in n8n) and not `{{ $json.execution.error.message }}` (which is for separate Error Trigger Workflows, not inline error pins).

**Memory de-duplication** (always on, server-side). StudioMeyer Memory's gatekeeper deduplicates writes on >95% content similarity automatically. If your workflow somehow fires twice on the same observation despite idempotency (clock skew, manual re-run), the second write is silently skipped. You can verify by checking `action: SKIPPED, similarity: 1` in the Memory response. This is server-side, no env var needed.

## Hard compatibility floor

**Minimum n8n version with CVE-2026-27493 fix:** >= 2.9.3 (stable channel) / >= 2.10.1 (latest / beta channel) / >= 1.123.22 (1.x LTS). CVE-2026-27493 is an unauthenticated RCE in Form nodes (CVSS 9.5). This template does not use Form nodes itself, but you should still upgrade for general security. The pre-activation check on n8n 2.15.0 was used to validate every node type-string in this template.

## Tech stack matrix

| Component | Version | Cost | Free tier | Required when |
|---|---|---|---|---|
| n8n | >= 2.10.1 (CVE-2026-27493 floor) | self-hosted free / Cloud $20/mo | n8n Cloud trial | always |
| n8n-nodes-studiomeyer-memory | >= 0.1.0 | free | n/a | always |
| StudioMeyer Memory | API key | EUR 0 / 29 / 49 | 200 credits (one per op) | always |
| OpenAI (default) | gpt-5-mini | $0.25 / 1M input + $2.00 / 1M output | $5 trial credit | provider = openai |
| Anthropic | claude-haiku-4-5 | $1 / 1M input + $5 / 1M output | $5 trial credit | provider = anthropic |
| Telegram (BotFather token) or WhatsApp Cloud API | latest stable | varies | trial available | always |

Telegram bots are free. Cloud API requires a verified Meta business account for WhatsApp.

## Credentials checklist

Before activation, create these credentials in n8n:

- [ ] **StudioMeyer Memory API** (`studioMeyerMemoryApi`). Get key at [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Test: any memory.search call returns `success: true`.
- [ ] **OpenAI API** (`openAiApi`) OR **Anthropic API** (`anthropicApi`). Get key at [platform.openai.com](https://platform.openai.com) / [console.anthropic.com](https://console.anthropic.com). Test: model list endpoint returns models.
- [ ] **Telegram** (Bot token from @BotFather). After activation, n8n registers the webhook automatically.
- [ ] **Telegram webhook secret token (recommended).** For pre-activation, expand the Telegram Trigger node's `additionalFields` and set `secretToken` to a strong random string. Then re-register the Telegram webhook with the same secret. Telegram sends `X-Telegram-Bot-Api-Secret-Token` on every webhook and the trigger validates it. Without this, the webhook is public-callable by anyone who guesses the URL.

## Live verification

The Memory pattern was smoke-tested live in [n8n.studiomeyer.io](https://n8n.studiomeyer.io) (n8n v2.15.0) against the production Memory backend at [memory.studiomeyer.io](https://memory.studiomeyer.io) on 2026-04-30:

| Execution | Path | Result |
|---|---|---|
| `445` | New customer → `Memory: Create Customer` → `Observe` + `Learn` | success, learn-id `7f08762c-ad67-493a-bdb3-fd6e7d626890` ADDED |
| `446` | Known customer (same key) → `Memory: Customer Dossier` (entity.open) → `Observe` + `Learn` | success, learn-id `0c45bd1f-254c-4b3b-9b7b-35a95729bb04` ADDED |

Both branches of the `Known Customer?` IF were exercised against the live Memory backend with a maintainer-side StudioMeyer Memory API credential. The full structural validation (25 nodes, 16 connections, 0 missing references, all node types recognized by n8n's pre-activation check) passed including the `@n8n/n8n-nodes-langchain.anthropic` type-string for the Anthropic Reply node.

Full end-to-end execution (with LLM Reply nodes firing real OpenAI / Anthropic calls and Telegram delivering the reply) requires user-supplied OpenAI or Anthropic plus Telegram credentials. To reproduce: import `workflow.json` into your own n8n, attach the four credentials (Memory API + OpenAI or Anthropic + Telegram bot), and either message your bot from Telegram or fire a synthetic update from inside the container with `docker exec <container> node -e "fetch('http://localhost:5678/webhook/<webhook-path>', {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({update_id: 1, message: {message_id: 1, from: {id: 1, username: 'test'}, chat: {id: 1}, text: 'hello'}})})"`.

## How this compares

The four memory layers a 2026 builder considers for an n8n bot:

| Feature | StudioMeyer Memory | Mem0 | Zep | Memori |
|---|---|---|---|---|
| Verified n8n custom node | Yes (this repo, npm provenance) | Community HTTP node | Community node | Third-party node |
| Reference templates ship | 8 templates in this repo | Reddit posts only | Some | None curated |
| Free tier | 200 credits (one per op) | 10K memories + 1K retrieval calls/mo | 1k credits/mo cloud + Graphiti OSS self-host | OSS self-host only |
| Bi-temporal `asOf` queries | Yes | Limited | Yes (via Graphiti) | No |
| Knowledge graph (entities + relations) | Native | Hybrid vector + graph | Native (Graphiti) | Vector only |
| Multi-tenant isolated by default | Yes | Manual config | Manual config | Self-host |
| EU hosting | Yes (Frankfurt, Hetzner) | US-default | US-default | Self-host |
| OAuth 2.1 + API key | Both | API key | API key | API key |

All four projects are open about their tradeoffs. The fastest comparison: wire each one to a throwaway Telegram bot for a day and see which response shape feels right for your use case.

## Related templates

- [01 - Voice Agent Cross-Session Memory](../01-voice-agent-cross-session-memory/) · same memory pattern over telephony (Vapi / Retell)
- [03 - Personal Assistant with Long-Term Memory](../03-personal-assistant-long-term-memory/) · single-user variant with intent classifier and tool use

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Part of the [StudioMeyer MCP Stack](../../README.md). Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues + ideas at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues).*
