<!-- studiomeyer-mcp-stack-banner:start -->
> **Part of the [StudioMeyer MCP Stack](https://studiomeyer.io)** · Built in Mallorca · ⭐ if you use it
<!-- studiomeyer-mcp-stack-banner:end -->

# Personal Assistant with Long-Term Memory (Multi-Provider)

> Tell it anything, ask it anything later. Notes, decisions, half-formed thoughts, they all become searchable. "What did I say about the redesign on Monday?" returns the right answer three weeks later. **Pick your LLM** (OpenAI default, Anthropic alternative).

![Cover](./cover.png)

## What this does

A single-user Telegram bot with three intents:

- **Note** (`/note <anything>` or just text): stored as a Memory learning. No LLM call. Reply: "Saved."
- **Ask** (`/ask <question>` or any non-slash text): searches Memory with recency-weighting, builds a context-aware prompt, lets Claude answer with citations to past entries.
- **Summary** (`/summary <topic>`): runs Memory's `synthesize` operation, clusters everything tagged with that topic into a coherent summary.

The killer feature is the third one. After three weeks of notes about a project, `/summary redesign` returns a paragraph that reads like you wrote it yourself, because it was generated from your own words.

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
[Detect Intent]                   ← slash-command parse, default = ask
        │
        ▼
   ┌──┤ Route by Intent ├──┐
   │      │      │      └─── fallback (treated as ask)
   ▼      ▼      ▼
 note   summary  ask
   │      │      │
   ▼      ▼      ▼
[Save] [Synthesize] [Search context]
   │      │      │
   ▼      ▼      ▼
[Reply] [Reply]  [Build Prompt]
                     │
                     ▼
            [Set Provider] → [Route by Provider]
                     ┌────────┴─────────┬─────────┐
                     ▼                  ▼         ▼
            [OpenAI Reply]    [Anthropic Reply]   fallback
            gpt-5-mini default  claude-haiku-4-5     │
            onError: continue   onError: continue    │
            │       │           │       │           │
          success error       success error         │
            │       │           │       │           │
            ▼       └───────────┘       └───────────┤
        [Normalize] ◄──────────────────────────[LLM Fallback Reply]
            │                                       │
            ├─ Telegram: Q&A Reply ◄─────────────────┤
            └─ Memory: Learn Q&A           └─ Memory: Learn Error
                                              (category: mistake)
```

The error branch is always wired. If OpenAI rate-limits, Anthropic returns 5xx, or the `provider` field is misconfigured, the user still gets a graceful reply and the failure lands in your knowledge graph as a `mistake` learning so you can spot patterns.

## Memory model

| Concept | Storage | Tags |
|---|---|---|
| Notes | `Learning`, category=insight, confidence=0.85 | `note, <user-label>` |
| Q&A pairs | `Learning`, category=insight, confidence=0.65 | `qa, <user-label>` |
| Summaries | Generated on-demand from existing learnings | n/a |

The asymmetric confidence (notes 0.85, Q&A 0.65) is deliberate. Notes are things you said with intent, Q&A pairs include your phrasing of a question and the LLM's answer, which is useful but slightly less authoritative.

All memory is scoped to `project: personal-assistant` so it doesn't mix with your work memory if you use Memory in multiple workflows.

## Setup

1. **Install the StudioMeyer Memory community node** (see repo README).

2. **Create a Telegram bot** via @BotFather, save the token.

3. **Add credentials in n8n:**
   - StudioMeyer Memory API → API Key from [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys).
   - Telegram → bot token from @BotFather.
   - OpenAI API (default provider) → key from [platform.openai.com](https://platform.openai.com), `gpt-5-mini` is the current default mini tier.
   - Anthropic API (alternative provider) → key from [console.anthropic.com](https://console.anthropic.com), `claude-haiku-4-5` is the current Haiku tier.

4. **Import the workflow** (`workflow.json`).

5. **Activate.** Telegram registers the webhook automatically.

6. **First test.** Open the bot in Telegram. Send `/note testing one two three`. You should get "Saved." back. Now send `did I test anything?`, the bot searches Memory and replies with the test note as a citation.

## Multi-provider switch

The workflow ships with two providers wired in parallel: OpenAI `gpt-5-mini` (default) and Anthropic `claude-haiku-4-5`. Pick one or run both behind a feature flag.

**Switch from OpenAI to Anthropic:**

1. Open the `Set Provider` node.
2. Change the `provider` field from `openai` to `anthropic`.
3. Save and re-test. The workflow now routes through `Anthropic Reply` and you get Anthropic's tone for Q&A.

**Add a third provider (Gemini, Mistral, local Ollama):**

1. Add a new Reply node using the appropriate credential type.
2. Add a new rule in `Route by Provider` that matches `provider == "gemini"` and routes to it.
3. On the new Reply node, set `On Error: Continue (Error Output)`. Connect main output 0 to `Normalize LLM Output` and main output 1 to `LLM Fallback Reply`.
4. Update `Normalize LLM Output` to handle the new provider's response shape (the existing code already handles OpenAI `choices[0].message.content` and Anthropic `content[0].text`, just append a third branch).
5. Set `provider` to `gemini` in `Set Provider` to test.

The error branch and Memory writes stay identical, you only add nodes, never edit the convergence point.

## Extending: add Google Calendar tool use

Here's how to give the assistant the ability to create calendar events. The pattern is identical for Gmail and Notion.

**Step 1.** Update the intent classifier. Replace the simple slash-command parser with a Claude-Haiku call that returns structured JSON:

```js
// Replace the contents of "Detect Intent" Code node
const body = $input.first().json;
const text = (body?.message?.text ?? '').trim();

// Use Anthropic API to classify
const response = await this.helpers.httpRequest({
  method: 'POST',
  url: 'https://api.anthropic.com/v1/messages',
  headers: {
    'x-api-key': $credentials.anthropicApi.apiKey,
    'anthropic-version': '2023-06-01',
    'content-type': 'application/json',
  },
  body: {
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 200,
    messages: [{
      role: 'user',
      content: `Classify this message into one of: note, ask, summary, calendar, email, notion. Reply with JSON {"intent": "...", "payload": "..."}.\n\nMessage: ${text}`,
    }],
  },
  json: true,
});

const parsed = JSON.parse(response.content[0].text);
return [{ json: { ...parsed, rawText: text, /* ... */ } }];
```

**Step 2.** Add a fourth branch to the Switch node for `intent: "calendar"`.

**Step 3.** In the calendar branch, add Google Calendar → Create Event. Map fields from the payload (Claude can return ISO timestamps and a title in its structured response).

**Step 4.** After the Calendar node, add Memory: Learn with `content: "Calendar event created: ${title} at ${start_time}"`. Now the assistant remembers it scheduled the event.

**Step 5.** Reply on Telegram with confirmation.

The full pattern: **classify intent with Haiku** → **execute via native n8n tool** → **persist to Memory** → **reply on Telegram**. Same skeleton works for Gmail (send), Notion (page create), Linear (issue create), or any service with an n8n node.

## Other extensions

**Daily morning brief.** Add a Schedule trigger (e.g. 7am) that runs `Memory: Synthesize` with `query: "yesterday"` and posts the cluster summary to your Telegram chat. You wake up to a recap of what you said the previous day.

**Voice notes.** Telegram bots can receive voice messages. Add a branch that: (a) downloads the voice file via `getFile`, (b) transcribes via the Anthropic Voice API or OpenAI Whisper, (c) treats the transcript as a `/note` payload. Now you can record thoughts hands-free while walking and ask about them later from your laptop.

**Multi-modal input.** When the user sends a photo, OCR it (via Claude's vision model directly on the image) and store the extracted text as a note. Receipts, whiteboard photos, screenshots, all become text-searchable.

## Cost notes

For a single user, ~50 messages/day mix of notes and questions:

| Component | Cost (Stand 2026-04) | Per-day cost |
|---|---|---|
| **StudioMeyer Memory** | EUR 0 / 29 / 49 per month | Free tier is 200 credits (one credit per op, enough for ~50 messages/day in dev). Pro at EUR 29/mo lifts the cap.. |
| **OpenAI gpt-5-mini** (default) | $0.25 / 1M input + $2.00 / 1M output | ~$0.014/day for 20 Q&A roundtrips (~500 in + 600 out tokens each) |
| **Anthropic claude-haiku-4-5** | $1 / 1M input + $5 / 1M output | ~$0.07/day for the same volume |
| Telegram Bot API | free | free |

**Worked example at 50 messages/day (30 notes + 20 Q&A):**

| Stack | Memory | LLM | Total /month |
|---|---|---|---|
| OpenAI gpt-5-mini | EUR 0 (free tier covers it) | ~$0.40/mo | **~$0.40/mo** |
| Anthropic claude-haiku-4-5 | EUR 0 | ~$2/mo | **~$2/mo** |

Past the 200-credit free tier (one credit per op) you need Pro at EUR 29/mo for the lifted monthly cap. Pro also unlocks the 3D knowledge-graph view at memory.studiomeyer.io/portal/memory/knowledge.

The error branch fires on LLM failures (rate limit, 5xx). It writes one extra Memory op (Learn Error) per failure. At a healthy 99.5% success rate this adds <0.5% to your bill.


## Common gotchas

- **Single-user assumption.** This template stores everything under one project namespace without per-user scoping. If you share the bot with multiple people, add the Telegram user ID to the project name (e.g. `personal-assistant-123456789`) so each person's notes stay private. Memory's tenant isolation is per-API-key, not per-user-within-an-API-key.
- **Slash command not recognized.** Telegram requires the bot to be registered with `/setcommands` via @BotFather for the slash-command UI to suggest them. Without that, users have to type the slash manually.
- **Markdown rendering.** Same caveat as Template 02, Telegram's Markdown is a subset. If the LLM generates a response with characters like `_` or `*` in code blocks, escape them or switch to `MarkdownV2`.
- **Memory not finding old notes.** Memory's search uses semantic + lexical scoring. Very short notes ("buy milk") have low semantic discriminability and may not surface for ambiguous queries. Mitigation: add a few extra words of context when noting (`/note shopping list: buy milk on Saturday`). The bot also auto-summarizes old notes when you run `/summary` on a topic, that often surfaces things plain search misses.
- **Concurrent saves and a search at the same time.** Memory's gatekeeper deduplicates writes within a 5-minute window. If you `/note X` then immediately `What did I just save?`, the search may not yet have indexed the embedding. Wait ~3 seconds or the search uses the lexical fallback (which still finds the new note in 99% of cases).
- **Anthropic node type-string.** The Anthropic Reply node uses `@n8n/n8n-nodes-langchain.anthropic` (the LangChain-vendored direct-API node), not `n8n-nodes-base.anthropic` (which does not exist in n8n core and produces "Unrecognized node type" on activation). The OpenAI counterpart is the core `n8n-nodes-base.openAi` (verified working in n8n 2.15.0). If you fork this template and a Reply branch fails to activate, double-check the type-string against your n8n version.
- **OpenAI vs Anthropic user-input field.** Both Reply nodes read the user-content from `{{ $json.question }}` (the field that the upstream `Build Prompt` Code node produces). Earlier revisions of this template had OpenAI reading `$json.messageText` which did not exist after `Build Prompt`, so OpenAI ran with empty user input. Both providers now use `$json.question`. If you fork and the OpenAI branch returns a non-sequitur, check the user-content expression first.
- **Anonymous Telegram messages.** Channel posts and forwarded messages without a sender have no `from.id`. The intent classifier falls back to `chat:<chatId>` so anonymous messages still get a stable user-label, with a hard `throw` when even chat id is missing. If you want strict per-sender separation, add an early IF that drops messages without `message.from.id`.

## Production patterns

Four patterns ship in `workflow.json` as actual nodes, three opt-in via env vars and one always-on error branch. A fifth pattern, Memory de-duplication, is server-side and needs no workflow node. The opt-in nodes pass through when their env var is unset, so the default import boots clean.

**Idempotency** (opt-in, `IDEMPOTENCY_ENABLED=1`). The `Idempotency Check` Code node holds a 5-minute in-memory window of seen `update_id` values and short-circuits duplicates. Telegram retries on 5xx so this catches double-fires without touching Memory or the LLM. For clustered n8n deployments, swap the `$getWorkflowStaticData` block for Redis `SET NX EX 300`. The node has the swap pattern in its comments.

**Rate limiting** (opt-in, `RATE_LIMIT_ENABLED=1`). The `Rate Limit` Code node caps each Telegram chat at 60 requests in a 5-minute sliding window. Without this, a misbehaving client or a leaked bot URL spikes your LLM bill. The map is bounded at 5 000 entries and evicts expired buckets when full. For real production loads, put rate limiting on a reverse proxy (Nginx `limit_req_zone`, Cloudflare WAF, Traefik) and keep this node as defense-in-depth.

**Webhook security** (opt-in, configured on the trigger). Telegram's `secretToken` mechanism is built into the Telegram Trigger node. Set `additionalFields.secretToken` to a strong random string and re-register the webhook with Telegram's `setWebhook?secret_token=...`. Telegram then sends `X-Telegram-Bot-Api-Secret-Token` on every call and the trigger validates it automatically. The optional `Verify Webhook` Code node downstream (toggled with `WEBHOOK_INTEGRITY_CHECK_ENABLED=1`) is the second defense layer: it rejects malformed payloads that lack the fields downstream nodes need. For non-Telegram triggers (WhatsApp, Slack, custom HTTP), replace the Telegram Trigger with a generic Webhook node and use a generic HMAC + length-guard pattern (see Template 01 Verify Webhook Code node for a working example).

**Error branches** (always on). Both LLM Reply nodes have `On Error: Continue (Error Output)` enabled. The error pin lands at `LLM Fallback Reply`, which builds a graceful customer-facing message and feeds two destinations: `Telegram: Q&A Reply` (so the user gets an answer) and `Memory: Learn Error` with `category: mistake, tags: [llm-error, <provider>]` (so you spot patterns in the knowledge graph). The `Route by Provider` fallback output (when the `provider` field is neither `openai` nor `anthropic`, e.g. typo) also lands at `LLM Fallback Reply`. The error syntax is `{{ $json.error.message }}`, not `{{ $error.message }}` (which does not exist in n8n) and not `{{ $json.execution.error.message }}` (which is for separate Error Trigger Workflows, not inline error pins).

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
| Telegram (BotFather token) | latest stable | varies | trial available | always |

Single-user only by default, see Common gotchas for multi-user scoping.

## Credentials checklist

Before activation, create these credentials in n8n:

- [ ] **StudioMeyer Memory API** (`studioMeyerMemoryApi`). Get key at [memory.studiomeyer.io/dashboard/keys](https://memory.studiomeyer.io/dashboard/keys). Test: any memory.search call returns `success: true`.
- [ ] **OpenAI API** (`openAiApi`) OR **Anthropic API** (`anthropicApi`). Get key at [platform.openai.com](https://platform.openai.com) / [console.anthropic.com](https://console.anthropic.com). Test: model list endpoint returns models.
- [ ] **Telegram** (Bot token from @BotFather). After activation, n8n registers the webhook automatically.
- [ ] **Telegram webhook secret token (recommended).** For pre-activation, expand the Telegram Trigger node's `additionalFields` and set `secretToken` to a strong random string. Then re-register the Telegram webhook with the same secret. Telegram sends `X-Telegram-Bot-Api-Secret-Token` on every webhook and the trigger validates it. Without this, the webhook is public-callable by anyone who guesses the URL.

## Live verification

This template's structural correctness was validated against n8n 2.15.0 in [n8n.studiomeyer.io](https://n8n.studiomeyer.io) during the v0.4.0-prep pass. Pre-activation check passed for all 26 nodes including the `@n8n/n8n-nodes-langchain.anthropic` type-string for the Anthropic Reply node and the `n8n-nodes-base.openAi` type-string for OpenAI. The Memory pattern (search + learn + synthesize) was smoke-tested in Template 02 against the same backend at memory.studiomeyer.io with executions 445 + 446 both green.

Full end-to-end execution requires user-supplied OpenAI or Anthropic plus Telegram credentials. To reproduce: import `workflow.json` into your own n8n, attach the four credentials (Memory API + OpenAI or Anthropic + Telegram bot), activate, and message the bot from Telegram. Or fire a synthetic webhook from inside the container with `docker exec <container> node -e "fetch('http://localhost:5678/webhook/<webhook-path>', {method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify({update_id: 1, message: {message_id: 1, from: {id: 1, username: 'test'}, chat: {id: 1}, text: '/note testing'}})})"`.

## How this compares

Memory layers a 2026 builder considers for an n8n personal-assistant bot:

| Feature | StudioMeyer Memory | Mem0 | Zep | Memori |
|---|---|---|---|---|
| Verified n8n custom node | Yes (this repo, npm provenance) | Community HTTP node | Community node | Third-party node |
| Reference templates ship | 8 templates in this repo | Reddit posts only | Some | None curated |
| Free tier | 200 credits (one per op) | 10K memories + 1K retrieval calls/mo | 1k credits/mo cloud + Graphiti OSS self-host | OSS self-host only |
| Bi-temporal `asOf` queries | Yes | Limited | Yes (via Graphiti) | No |
| Knowledge graph (entities + relations) | Native | Hybrid vector + graph | Native (Graphiti) | Vector only |
| `synthesize` operation (cluster summary) | Native | No | No | No |
| Multi-tenant isolated by default | Yes | Manual config | Manual config | Self-host |
| EU hosting | Yes (Frankfurt, Hetzner) | US-default | US-default | Self-host |
| OAuth 2.1 + API key | Both | API key | API key | API key |

The `synthesize` operation is the differentiator for a personal-assistant template. `/summary redesign` clusters every learning tagged `redesign` across weeks of notes into a coherent paragraph, no other backend in the comparison ships that as a first-class API.

All four projects are open about their tradeoffs. The fastest comparison: wire each one to a throwaway Telegram bot for a day and see which response shape feels right for your use case.

## Related templates

- [01 - Voice Agent Cross-Session Memory](../01-voice-agent-cross-session-memory/) · same memory pattern over telephony (Vapi / Retell)
- [02 - AI Customer Support with History](../02-customer-support-with-history/) · multi-customer chat variant

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Part of the [StudioMeyer MCP Stack](../../README.md). Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues + ideas at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues).*
