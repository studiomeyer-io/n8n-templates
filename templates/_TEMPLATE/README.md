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
[Trigger]
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
[Convergence]
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

## Multi-Provider Switch

The workflow has a `Set Provider` node followed by a `Route by Provider` switch. Default value is `openai`. Change to `anthropic` (or add your own branch) without rebuilding the rest of the flow. Both branches converge in `Normalize LLM Output` which extracts reply text from either provider's response shape.

To add a third provider (Gemini, Mistral, local Ollama):

1. Open `Route by Provider`, add a third rule matching `provider == "gemini"`.
2. Drag in the corresponding LLM node (or HTTP Request for self-hosted).
3. Connect it back to `Normalize LLM Output`.
4. The Code node already handles common shapes; if the new provider returns something exotic, add one more `else if` branch.

## Extending

**<First extension idea>.** One paragraph in flowing prose explaining what to add and where. Reference specific node names from the workflow.

**<Second extension idea>.** Same shape.

**<Third extension idea>.** Same shape.

## Cost notes

Per execution (assuming average payload):

| Component | Approx cost |
|---|---|
| 2× Memory operations | < $0.001 |
| 1× LLM call (~500 tokens) | ~$0.0005 |
| 1× async memory write | < $0.001 |
| **Total per execution** | **~$0.002** |

At <typical-volume> / month, expect ~$<X> / month.

## Common gotchas

- **<First gotcha>.** Explain what goes wrong, why, and how to fix it. One paragraph.
- **<Second gotcha>.** Same.
- **<Third gotcha>.** Same.

## Related templates

- [01 - Voice Agent Cross-Session Memory](../01-voice-agent-cross-session-memory/), voice provider variant
- [02 - AI Customer Support with History](../02-customer-support-with-history/), multi-customer chat variant
- [03 - Personal Assistant with Long-Term Memory](../03-personal-assistant-long-term-memory/), single-user variant with tool use

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Part of the [StudioMeyer MCP Stack](../../README.md). Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues / questions at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues).*
