# Sample Payloads

> Synthetic but realistic webhook payloads for the three Tier-1 templates. Use them to smoke-test before activating the workflow against a real provider account. See [PRODUCTION_CHECKLIST.md](../PRODUCTION_CHECKLIST.md) for the surrounding pre-flight steps.

## What's here

| File | For template | Provider event |
|---|---|---|
| [`vapi-end-of-call.json`](./vapi-end-of-call.json) | 01 - Voice Agent | Vapi `end-of-call-report` |
| [`retell-call-ended.json`](./retell-call-ended.json) | 01 - Voice Agent | Retell `call_ended` |
| [`telegram-message.json`](./telegram-message.json) | 02 - Customer Support, 03 - Personal Assistant | Telegram Bot API `Update` |

All three payloads share the same fictional customer (Maria Schmidt, `+49170555444`) so you can chain them across templates and watch the same caller persist across channels.

## How to use

### Test in n8n's UI

1. Open the workflow in your n8n instance.
2. Click the Webhook trigger node.
3. Click **Listen for test event** (top right of the node panel).
4. Copy the test-webhook URL n8n shows you.
5. POST the matching payload:

```bash
curl -X POST "https://your-n8n.example.com/webhook-test/<your-id>" \
  -H "Content-Type: application/json" \
  -d @examples/vapi-end-of-call.json
```

6. Watch the execution graph in n8n. Each node should turn green. Memory writes appear at [memory.studiomeyer.io/portal/memory/keys](https://memory.studiomeyer.io/portal/memory/keys) under Activity.

### Test signed payloads (Template 01)

The Verify Webhook Code node in Template 01 expects an `x-vapi-signature` or `x-retell-signature` header containing HMAC-SHA256 of the raw body, hex-encoded, signed with `VAPI_SIGNING_SECRET` or `RETELL_SIGNING_SECRET`.

Compute and send the signature with one curl call:

```bash
SECRET=your-vapi-signing-secret
PAYLOAD=$(cat examples/vapi-end-of-call.json)
SIG=$(echo -n "$PAYLOAD" | openssl dgst -sha256 -hmac "$SECRET" | awk '{print $2}')
curl -X POST "https://your-n8n.example.com/webhook-test/<your-id>" \
  -H "Content-Type: application/json" \
  -H "x-vapi-signature: $SIG" \
  --data-raw "$PAYLOAD"
```

If `VAPI_SIGNING_SECRET` is unset on the n8n host, the node passes through and the signature is ignored. With the secret set, a wrong or missing signature makes the workflow throw a HMAC verification error - which is the point.

### Test idempotency (all templates)

Send the same payload twice within five minutes with `IDEMPOTENCY_ENABLED=1`. The first run executes fully. The second run short-circuits at the Idempotency Check node and returns no items downstream.

### Test the Telegram secret_token

After you call `setWebhook` with `secret_token=<random>` and configure the same value as the `secretToken` parameter on the Webhook node:

```bash
curl -X POST "https://your-n8n.example.com/webhook-test/<your-id>" \
  -H "Content-Type: application/json" \
  -H "X-Telegram-Bot-Api-Secret-Token: <random>" \
  -d @examples/telegram-message.json
```

A wrong or missing token produces an n8n 401, never reaches the workflow.

## Caveats

- The payloads are **synthetic**. The Vapi/Retell/Telegram schemas evolve. If a field shape changes upstream and your workflow breaks, check the provider's current schema docs (links inside each `_comment` field).
- The `_comment` field is non-standard JSON and a real provider will not send it. n8n's Webhook node ignores unknown top-level keys, so the field is harmless during testing. Remove it before production replays if you care about strictness.
- The recording URLs and IDs are fictional and will not resolve.
- The transcripts assume English voice and German Telegram messages. Edit them if your stack expects different.

## Adding more samples

If you build a template that needs a new provider payload (Slack `event_callback`, WhatsApp Cloud API, Stripe `checkout.session.completed`), drop the JSON here with a `_comment` field linking the provider docs and add a row to the table above.
