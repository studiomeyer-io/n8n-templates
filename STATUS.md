# Template Status

> **Single source of truth for what's hardened, what's developer-preview, and where the distribution push stands.** Updated per release. Last update: 2026-05-01 (v0.4.0-prep).

## Per-template status

| # | Template | Status | What's wired | What's pending |
|---|---|---|---|---|
| 01 | [Voice Agent Cross-Session Memory](./templates/01-voice-agent-cross-session-memory/) | **Hardened** (v0.4.0-prep) | All 4 production patterns as nodes (HMAC verify with `VAPI_SIGNING_SECRET` / `RETELL_SIGNING_SECRET`, rate limit, idempotency on `callId`, error branch with LLM Fallback Reply + Memory: Learn Error). Real `Has Phone?` IF branches anonymous calls away from Memory. Anthropic node type-string fixed to `@n8n/n8n-nodes-langchain.anthropic`. | End-to-end smoke test against a Vapi or Retell account. Maintainer ships test trace once provider account is provisioned. |
| 02 | [AI Customer Support with History](./templates/02-customer-support-with-history/) | **Hardened** (v0.4.0-prep) | All 4 production patterns as nodes. Anthropic type-string fixed. Customer-key extractor with email > tg-id > chat-id fallback chain. Live-verified against memory.studiomeyer.io: executions 445 + 446 both green, both branches of Known Customer? IF exercised. | Distribution push (held until 01 has end-to-end smoke). |
| 03 | [Personal Assistant with Long-Term Memory](./templates/03-personal-assistant-long-term-memory/) | **Hardened** (v0.4.0-prep) | All 4 production patterns as nodes. Anthropic type-string fixed. OpenAI prompt-bug fixed (was reading `$json.messageText` which did not exist after Build Prompt). LLM Fallback Reply discriminates router-fallback vs LLM-error to avoid leaking `systemPrompt` into the audit trail. Detect Intent userLabel fallback chain (no more `user-undefined`). Structurally validated against n8n 2.15.0. | Distribution push (held until 01 has end-to-end smoke). |

## Repo-level status

| Item | Status |
|---|---|
| MIT License + CONTRIBUTING + SECURITY + COC + ECOSYSTEM | Done |
| GitHub Actions CI (workflow.json validation, em-dash guard, forbidden-keys check, credential-leak scan) | Done |
| Internal quality standard for voice + structure + quality gate | Done (maintainer-side) |
| `templates/_TEMPLATE/` skeleton for new templates | Done |
| Skill `~/.claude/skills/n8n-template-create/` for replication recipe | Done |
| `n8n-nodes-studiomeyer-memory` custom node v0.1.0 on npm with provenance | Done (separate repo) |
| Cover images (Flux 2 Max, 1216x640, navy + gold) for all 3 Tier-1 templates | Done |
| Comparison tables vs Mem0 / Zep / Memori in each template README | Done |
| Hard compatibility floor (n8n 2.10.1+ for CVE-2026-27493) declared | Done |
| Honest production-readiness framing ("production-pattern hardened, not one-click prod deploy") | Done |
| Distribution push (n8n.io/workflows + awesome-n8n-templates + dev.to + Reddit + LinkedIn) | **Held** until Template 01 ships an end-to-end voice smoke test |

## What "hardened" means

Each Tier-1 template has been through the same six-phase pass:

1. **Audit** against the internal quality checklist (12 mandatory README sections, sticky note color palette 5/6/7, no em-dashes, no forbidden top-level keys).
2. **v0.4.0 production patterns** as actual opt-in nodes in `workflow.json`: Verify Webhook (HMAC where applicable), Rate Limit, Idempotency Check (gated by env vars, default-off, pass-through when unset). Plus always-on Error-Output branch on both LLM Reply nodes wired to LLM Fallback Reply + Memory: Learn Error.
3. **README polish** to Reddit-readable level: ASCII architecture diagram, multi-provider switch section, concrete cost numbers with current 2026 LLM pricing, 5+ extending patterns, comparison table vs Mem0/Zep/Memori, live-verification table or honest "structurally validated" note.
4. **Cover image** verified Brand-conform (1216x640 navy + gold).
5. **Quality Gate** parallel pass with three dedicated agents (Critic for bugs + Architect for repo-convention compliance + Research for current 2026 standards). All AMBER findings fixed before commit. Plus structural validation via the live n8n.studiomeyer.io pre-activation check (catches "Unrecognized node type" errors that import-time validation alone misses).
6. **Commit** with detailed message + CHANGELOG entry, push to main.

A template is "hardened" when all six phases are green and the internal self-check passes.

## What's NOT yet done

- **End-to-end smoke test of Template 01.** Requires a Vapi or Retell test account. The maintainer has the account provisioning on the next sprint. The Memory pattern is proven via Template 02 (executions 445 + 446 both green) which uses the same backend, but the Vapi/Retell webhook ingestion path itself has not been smoke-tested live against a real call. This is the gating item for distribution push.
- **`n8n-nodes-base.openAi` vs `@n8n/n8n-nodes-langchain.openAi` decision.** All three templates currently use `n8n-nodes-base.openAi` (verified working in n8n 2.15.0 pre-activation check). Newer n8n versions also expose `@n8n/n8n-nodes-langchain.openAi`. Templates may migrate to the LangChain variant in v0.5.0 once we verify it works across the n8n version matrix.
- **Distribution push** to n8n.io/workflows, awesome-n8n-templates PR, dev.to long-form articles, Reddit r/n8n + r/AI_Agents, LinkedIn DACH PDF carousels. Held until Template 01 end-to-end smoke is documented.

## Public claims discipline

A repo-internal rule: **public repo description never claims more than the weakest production-ready path in the repo claims.**

GitHub repo description: "Production-pattern hardened n8n workflow templates with cross-session memory."
Top-level README: "Production-pattern hardened, not a one-click production deploy yet" with explicit per-template status.
This `STATUS.md`: ground truth, updated per release.

If you find a discrepancy between any of these, file an issue. The README and STATUS.md are the source of truth, the GitHub description tracks them.

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io).*
