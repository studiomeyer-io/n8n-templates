# Contributing

Thanks for considering a contribution. The bar for new templates is high — each one represents StudioMeyer Memory in front of thousands of n8n users on n8n.io and through the StudioMeyer marketing channels.

## What we accept

A template is a strong candidate when it:

- Solves a concrete workflow problem where memory makes the difference between a useful bot and a dumb one.
- Demonstrates at least one meaningful Memory operation (search, learn, entity-create, entity-search, decide, synthesize).
- Has a clear primary trigger (webhook, schedule, chat) and a clear primary outcome (reply, notify, store, sync).
- Works end-to-end after the user fills in their credentials. No "TODO finish this" branches.

A template is **not** a good fit when it:

- Is a thin wrapper around a single Memory call without any orchestration.
- Requires a paid third-party service most users won't have (e.g. an exotic CRM with no free tier).
- Replicates something already in this repo with cosmetic changes.

## Folder layout

```
templates/NN-descriptive-slug/
├── workflow.json     # exported from n8n, hand-cleaned (see below)
├── README.md         # mandatory (see template below)
└── cover.md          # cover-image spec for n8n.io
```

The slug starts with a two-digit prefix matching the position in [README.md](./README.md). Tier 1 = `01-`–`03-`, Tier 2 = `04-`–`06-`, Tier 3 = `07-`–`08-`.

## workflow.json checklist

Before committing:

1. **Strip credentials.** Open the JSON and remove any `credentials` blocks that point to a real credential ID. The user's import flow re-attaches their own credentials.
2. **Strip versionIds + ids.** Set every node `id` to a stable string (e.g. `"voice-1"`, `"voice-2"`) instead of a UUID. n8n regenerates them on import.
3. **Remove `pinData`.** Test data should not ship.
4. **Set `active: false`.** Imports should never go live silently.
5. **Sanitize webhook paths.** Replace any random path segments with descriptive defaults (e.g. `vapi-callback`).
6. **Add `>> SET ME <<` markers.** Every value the user must change (webhook URL, model name, tokens, sender ID) must be obvious. Use either a sticky note or a `// SET ME:` comment in code nodes.
7. **Test the import path yourself.** Import the JSON into a fresh n8n instance, fill in the placeholders, and run a real test transaction.

## README.md template

Every template's README must contain these sections in this order:

```markdown
# Template Title

> One-sentence value prop. What does the user get?

## What this does

Two-paragraph plain-English explanation of the data flow. Mention the
external services (Vapi, WhatsApp, etc.) and the memory operations used.

## Architecture

ASCII diagram of the flow. Use the same style as the other templates.

## Memory model

How memory is keyed (caller phone, customer email, telegram-chat-id, etc.),
which observations get written, and which entity types are created.

## Setup

Numbered steps for credentials, env-vars, webhook registration, and a
first test call.

## Extending

2-3 concrete extension ideas with one-paragraph guidance each.

## Cost notes

Rough order-of-magnitude estimate per execution (memory ops + LLM tokens).
```

## Cover image

Each template needs a cover image for the n8n.io submission. Recommended size: 1200×630, dark background, three icons (trigger source, memory, LLM), and the template title in bold. We use Flux for generation; the prompt and resulting URL go into `cover.md`.

## Commit + PR

- One template per PR.
- Branch name `template/NN-slug`.
- Commit message: `feat(templates): add NN <title>`.
- PR description must include a screenshot of a successful test run from your own n8n instance.

## Review

A maintainer will:

1. Import your `workflow.json` into a clean n8n instance.
2. Run one end-to-end test against `memory.studiomeyer.io`.
3. Read the README for clarity.

If everything checks out, we merge, tag the next minor version, and submit the template to n8n.io within a week.
