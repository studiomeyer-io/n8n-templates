# Production Checklist

> **What to flip before you point a real workflow at a real user.** None of these templates are production-ready out of the box. They boot clean on default import (good for evaluation), but the production-patterns are opt-in via env vars (better for "ship it"). This file is the checklist for that flip.
>
> Last update: 2026-05-01.

## TL;DR

| Layer | What to do | Hard requirement? |
|---|---|---|
| n8n version | upgrade to >= 2.9.3 (stable) / >= 2.10.1 (latest) / >= 1.123.22 (LTS) - CVE-2026-27493 fix | yes |
| Node.js builtins | set `NODE_FUNCTION_ALLOW_BUILTIN=crypto` if self-hosted (Template 01 needs it) | yes for self-host |
| Webhook signing | set `VAPI_SIGNING_SECRET` / `RETELL_SIGNING_SECRET` / Telegram `secretToken` | yes |
| Idempotency | set `IDEMPOTENCY_ENABLED=1` | recommended |
| Rate limiting | set `RATE_LIMIT_ENABLED=1` | recommended |
| Webhook integrity | set `WEBHOOK_INTEGRITY_CHECK_ENABLED=1` (T02 + T03) | recommended |
| Cluster + scale | swap in-memory dedup for Redis SET NX | yes if running >1 n8n worker |
| Monitoring | wire executions to Sentry / Slack / Telegram failure channel | yes |
| Secrets management | move env vars from `.env` files to a secret manager | yes for production |
| Backup | export workflow.json to git after every edit | recommended |

If even one of those rows is unchecked, you are running a developer-preview deployment. Be honest with yourself about that.

## Per-template env vars

### Template 01 - Voice Agent Cross-Session Memory

```bash
# Webhook HMAC verification (one or both)
VAPI_SIGNING_SECRET=<random 32-byte hex>          # configure same value in Vapi dashboard
RETELL_SIGNING_SECRET=<random 32-byte hex>        # configure same value in Retell dashboard

# Production patterns (opt-in, default-off)
IDEMPOTENCY_ENABLED=1                             # dedup retries on callId
RATE_LIMIT_ENABLED=1                              # 60 calls / 5 min / phone

# n8n Code-node builtin allowlist (self-host only)
NODE_FUNCTION_ALLOW_BUILTIN=crypto                # required for HMAC verify Code node
```

**Why `NODE_FUNCTION_ALLOW_BUILTIN=crypto` matters:** the Verify Webhook Code node calls `require('crypto')` for HMAC-SHA256 + `timingSafeEqual`. Self-hosted n8n restricts Node.js builtin imports in Code nodes by default. Without this env var the node throws `Cannot find module 'crypto'` and the workflow rejects every signed webhook. **n8n Cloud's allowlist is not publicly documented for individual builtins** - verify in your Cloud instance by adding a one-line `require('crypto')` test in a throwaway Code node before flipping HMAC verification on in production. See [n8n Docs - Modules in Code node](https://docs.n8n.io/hosting/configuration/configuration-examples/modules-in-code-node/).

### Template 02 - AI Customer Support with History

```bash
# Telegram secret token (header check on incoming webhooks)
# Set this when you call setWebhook on the Telegram bot:
#   curl -X POST "https://api.telegram.org/bot<TOKEN>/setWebhook" \
#     -d "url=https://your-n8n.example.com/webhook/telegram-cs" \
#     -d "secret_token=<random>"
# Then set the same value as the secretToken parameter on the Webhook node.

# Production patterns (opt-in, default-off)
WEBHOOK_INTEGRITY_CHECK_ENABLED=1
RATE_LIMIT_ENABLED=1
IDEMPOTENCY_ENABLED=1
```

### Template 03 - Personal Assistant Long-Term Memory

```bash
# Same Telegram secret_token pattern as T02

# Production patterns (opt-in, default-off)
WEBHOOK_INTEGRITY_CHECK_ENABLED=1
RATE_LIMIT_ENABLED=1
IDEMPOTENCY_ENABLED=1
```

### All templates

```bash
# StudioMeyer Memory credential (created in n8n Credentials, not env)
# Get a key at studiomeyer.io/portal/login -> "Free Memory testen" (200 free credits, no card)
# Or paste an OAuth 2.1 PKCE access token from memory.studiomeyer.io/.well-known/oauth-authorization-server

# LLM credentials (one or both, created in n8n Credentials)
# OpenAI: platform.openai.com/api-keys
# Anthropic: console.anthropic.com/settings/keys
```

## Cluster + scale

`$getWorkflowStaticData('global')` is in-memory and per-instance. Two n8n workers behind a load balancer do not share it. Two concurrent executions on the same worker can both pass the dedup check in the millisecond window between read and write.

**If you run more than one n8n worker, swap to Redis:**

```javascript
// Idempotency Code node - Redis variant
// Replace the $getWorkflowStaticData block with:
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);  // redis://host:6379
const key = `idem:vapi:${callId}`;
const wasSet = await redis.set(key, '1', 'EX', 300, 'NX');  // 5-min TTL, only if not exists
if (!wasSet) {
  return [];  // duplicate, drop
}
return [{ json: $input.first().json }];
```

`SET NX EX 300` is atomic and cluster-aware. Same pattern for Rate Limit (use `INCR + EXPIRE` instead). Add `NODE_FUNCTION_ALLOW_EXTERNAL=ioredis` to the n8n env if self-host.

## Reverse-proxy rate limiting (preferred)

The in-flow Rate Limit Code node is defense-in-depth. For real production loads put rate limiting at the edge:

- **Nginx:** `limit_req_zone $binary_remote_addr zone=n8n:10m rate=20r/s;` plus `limit_req zone=n8n burst=40 nodelay;` in the location block
- **Cloudflare:** WAF Custom Rule on the webhook path
- **Traefik:** [`RateLimit` middleware](https://doc.traefik.io/traefik/middlewares/http/ratelimit/)

These are atomic, cluster-aware, faster, and don't burn n8n execution time.

## Monitoring

- **Failed executions:** wire n8n's Error Trigger Workflow to a notification channel (Slack, Telegram, email). Document: [n8n Error Workflows](https://docs.n8n.io/flow-logic/error-handling/).
- **Memory writes:** every `Memory: Learn Error` entry uses `category: mistake` + provider tag. A weekly synthesis (`memory.synthesize` with `query: 'category:mistake last week'`) surfaces patterns.
- **LLM cost:** OpenAI + Anthropic both expose usage dashboards. Alert at 80% of monthly cap.
- **Webhook 429 / 5xx:** the voice provider's own retry log is your second source of truth. Vapi shows delivery status per `end-of-call-report`, Retell similar.

## Secrets management

`.env` files are fine for development. For production:

- **n8n Cloud:** set env vars in the project settings UI
- **Self-host Docker:** Docker secrets or a sidecar like `vault-agent`
- **Self-host bare-metal:** systemd `EnvironmentFile=` pointing at a chmod-600 file owned by the n8n user
- Rotate `*_SIGNING_SECRET` and StudioMeyer Memory API keys quarterly

## Smoke tests before activation

The [`examples/`](./examples/) folder ships sample provider payloads:

- `examples/vapi-end-of-call.json` - Vapi `end-of-call-report` shape
- `examples/retell-call-ended.json` - Retell `call_ended` shape
- `examples/telegram-message.json` - Telegram `Update` shape

To smoke-test before flipping to production:

1. Import the template, fill the SET-ME markers, leave the workflow inactive
2. In the Webhook node, click *Listen for test event*
3. Send the matching payload via curl:
   ```bash
   curl -X POST <test-webhook-url> \
     -H "Content-Type: application/json" \
     -d @examples/vapi-end-of-call.json
   ```
4. Watch the execution in n8n. Verify Memory writes happened (check `memory.studiomeyer.io/portal/memory/keys` -> Activity).
5. Repeat with `IDEMPOTENCY_ENABLED=1` set and the same payload twice. Second run should short-circuit.
6. Repeat with a malformed signature header. Should fail with the HMAC error.

When all six smoke tests pass with the production env vars set, the template is ready to activate.

## Pre-launch checklist (copy-paste for your runbook)

```
[ ] n8n version >= 2.9.3 (stable) OR >= 2.10.1 (latest) OR >= 1.123.22 (LTS)
[ ] NODE_FUNCTION_ALLOW_BUILTIN=crypto set (self-host + T01)
[ ] *_SIGNING_SECRET / Telegram secretToken configured both sides
[ ] IDEMPOTENCY_ENABLED=1
[ ] RATE_LIMIT_ENABLED=1
[ ] WEBHOOK_INTEGRITY_CHECK_ENABLED=1 (T02 + T03)
[ ] Redis configured if more than one n8n worker
[ ] Reverse-proxy rate limit in place
[ ] Error Trigger Workflow wired to notification channel
[ ] All three smoke tests in examples/ pass against the inactive workflow
[ ] Memory API key rotated less than 90 days ago
[ ] Workflow exported to git after every edit
[ ] Cost alerts set on OpenAI / Anthropic at 80% of monthly cap
[ ] Vapi or Retell test number wired and one real call placed end-to-end
[ ] On-call documented (who fixes a 3am LLM 429)
```

If a row is unchecked, the deployment is developer-preview, not production. Be honest with the next person who reads your runbook.

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues + ideas at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues).*
