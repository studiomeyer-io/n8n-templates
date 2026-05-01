# Template Status

> **Single source of truth for what's hardened, what's developer-preview, and where the distribution push stands.** Updated per release. Last update: 2026-05-01 (v0.5.0-prep, Sprint B + C build complete).

## Per-template status

| # | Template | Status | What's wired | What's pending |
|---|---|---|---|---|
| 01 | [Voice Agent Cross-Session Memory](./templates/01-voice-agent-cross-session-memory/) | **Hardened** (v0.4.0) | All 4 production patterns as nodes (HMAC verify with `VAPI_SIGNING_SECRET` / `RETELL_SIGNING_SECRET`, rate limit, idempotency on `callId`, error branch with LLM Fallback Reply + Memory: Learn Error). Real `Has Phone?` IF branches anonymous calls away from Memory. Anthropic node type-string fixed to `@n8n/n8n-nodes-langchain.anthropic`. JSON.stringify guard on Respond-to-Webhook (S955 fix prevents JSON-injection if caller transcript contains double-quote). Pricing updated to gpt-5.4-mini at $0.75/$4.50 per 1M (Stand 2026-05). | End-to-end smoke test against a Vapi or Retell account. Maintainer ships test trace once provider account is provisioned. |
| 02 | [AI Customer Support with History](./templates/02-customer-support-with-history/) | **Hardened** (v0.4.0) | All 4 production patterns as nodes. Anthropic type-string fixed. Customer-key extractor with email > tg-id > chat-id fallback chain. `isLlmError` discriminator added to LLM Fallback Reply (S955 fix prevents systemPrompt + customer-dossier from leaking into Memory: Learn Error audit trail when Route by Provider hits the typo-fallback). Pricing updated to gpt-5.4-mini. Live-verified against memory.studiomeyer.io: executions 445 + 446 both green, both branches of Known Customer? IF exercised. | Distribution push (held until 01 has end-to-end smoke). |
| 03 | [Personal Assistant with Long-Term Memory](./templates/03-personal-assistant-long-term-memory/) | **Hardened** (v0.4.0) | All 4 production patterns as nodes. Anthropic type-string fixed. OpenAI prompt-bug fixed (was reading `$json.messageText` which did not exist after Build Prompt). LLM Fallback Reply discriminates router-fallback vs LLM-error to avoid leaking `systemPrompt` into the audit trail. Detect Intent userLabel fallback chain (no more `user-undefined`). Duplicate connection at index 3 of Route by Intent removed (S955 fix prevents potential double-trigger of Memory: Search Context if Switch outputMode is changed to all-matches). Pricing updated to gpt-5.4-mini. Structurally validated against n8n 2.15.0. | Distribution push (held until 01 has end-to-end smoke). |
| 04 | [Restaurant Stammgast-Bot](./templates/04-restaurant-stammgast-bot/) | **Hardened** (v0.5.0-prep) | T02-pattern adapted to phone-customer-key. All 4 production patterns. Telegram trigger + secretToken HMAC. Customer-key extractor: phone (E.164 regex), Telegram contact-share, Telegram user-id, chat-id fallback chain. Memory: entity.search → entity.open / create → observe + learn. System prompt customised for restaurant-host context. Cover image generated (Flux 2 Max). Live-import + pre-activation-check passed against n8n.studiomeyer.io v2.15.0. | End-to-end test with a real Telegram bot in a restaurant pilot. |
| 05 | [Tourist-Bot Repeat-Visitor](./templates/05-tourist-bot-repeat-visitor/) | **Hardened** (v0.5.0-prep) | T02-pattern adapted to web-chat webhook + sessionId / fingerprint identity. Generic Webhook trigger with `rawBody: true` + `TOURIST_WEBHOOK_SIGNING_SECRET` HMAC. Memory entity for visitor-session. Recency-weighted system prompt for re-engagement. Cover image generated. Live-import + pre-activation-check passed. | End-to-end test with a real web-chat widget on a tourism site. |
| 06 | [Lead-Qualifier with BANT+I and Pipedrive](./templates/06-lead-qualifier-pipedrive/) | **Hardened** (v0.5.0-prep) | T02-pattern + Pipedrive HTTP Request node. Form Webhook trigger + `LEAD_FORM_SIGNING_SECRET` HMAC. BANT+I (Budget / Authority / Need / Timeline / Intent) classification via strict-JSON LLM call. Pipedrive deal creation per qualification bucket (hot / warm / cold) via env-var-configured stage IDs. Cover image generated. Live-import + pre-activation-check passed. | End-to-end test with a real Pipedrive instance + form submission. Decision: HubSpot / Salesforce variants in v0.6 if requested. |
| 07 | [Meeting-Bot Cross-Meeting Continuity](./templates/07-meeting-bot-cross-meeting-continuity/) | **Hardened** (v0.5.0-prep) | T02-pattern + Slack incoming-webhook + participant-set-hash entity-keying (SHA-256 first 16 hex). Auto-detection of Fathom / Otter / Granola / generic webhook shapes. Cross-meeting synthesis prompt that references last 5 meetings with the same participant set. `Webhook Acknowledge` parallel branch sends fast 200-OK to provider while LLM summary work happens async. Cover image generated. Live-import + pre-activation-check passed. | End-to-end test with a Fathom or Otter or Granola account. |
| 08 | [Mem0 / Zep Migration to StudioMeyer Memory](./templates/08-mem0-zep-migration/) | **Developer-preview** (v0.5.0-prep) | Pure ETL workflow. Manual Trigger or HTTP Trigger. Mem0 + Zep source-API auto-detection in `Validate + Configure`. SplitInBatches loop (10 records / batch). Memory entity-create + observe + learn per record with idempotency tags (source-system + import-date + user-id). Migration Report at end with success / error counts. NO LLM provider required (pure ETL). Cover image generated. Live-import + pre-activation-check passed. | Real-world migration test against a Mem0 or Zep account with at least 100 records. Pagination logic for >1000 records is best-effort (tested for first-page only). Re-keying follow-up workflow documented but not shipped as separate template yet. |

## Repo-level status

| Item | Status |
|---|---|
| MIT License + CONTRIBUTING + SECURITY + COC + ECOSYSTEM | Done |
| GitHub Actions CI (workflow.json validation, em-dash guard, forbidden-keys check, credential-leak scan) | Done |
| Internal quality standard for voice + structure + quality gate | Done (maintainer-side) |
| `templates/_TEMPLATE/` skeleton on v0.4.0 standard | Done (Sections 10b/10c/10d/10e added in S955) |
| Skill `~/.claude/skills/n8n-template-create/` for replication recipe | Done |
| `n8n-nodes-studiomeyer-memory` custom node v0.1.0 on npm with provenance | Done (separate repo) |
| Cover images (Flux 2 Max, 1216x640, navy + gold) for all 8 templates | Done |
| Comparison tables vs Mem0 / Zep / Memori in each template README | Done |
| Hard compatibility floor (n8n 2.10.1+ for CVE-2026-27493) declared | Done |
| Honest production-readiness framing ("production-pattern hardened, not one-click prod deploy") | Done |
| Pricing in all 8 READMEs synced to Stand 2026-05 (gpt-5.4-mini at $0.75/$4.50) | Done (S955) |
| Distribution push (n8n.io/workflows + awesome-n8n-templates + dev.to + Reddit + LinkedIn) | **Held** until Template 01 ships an end-to-end voice smoke test |

## What "hardened" means

Each Tier-1 + Tier-2 + Tier-3 template has been through the same six-phase pass:

1. **Audit** against the internal quality checklist (12 mandatory README sections + 4 sub-sections, sticky note color palette 5/6/7, no em-dashes, no forbidden top-level keys).
2. **v0.4.0 production patterns** as actual opt-in nodes in `workflow.json`: Verify Webhook (HMAC where applicable), Rate Limit, Idempotency Check (gated by env vars, default-off, pass-through when unset). Plus always-on Error-Output branch on both LLM Reply nodes wired to LLM Fallback Reply + Memory: Learn Error.
3. **README polish** to Reddit-readable level: ASCII architecture diagram, multi-provider switch section, concrete cost numbers with current 2026 LLM pricing, 4+ extending patterns, comparison table vs Mem0/Zep/Memori, live-verification table or honest "structurally validated" note.
4. **Cover image** verified Brand-conform (1216x640 navy + gold via Flux 2 Max).
5. **Quality Gate** parallel pass with three dedicated agents (Critic for bugs + Architect for repo-convention compliance + Research for current 2026 standards). All AMBER findings fixed before commit. Plus structural validation via the live n8n.studiomeyer.io pre-activation check (catches "Unrecognized node type" errors that import-time validation alone misses).
6. **Commit** with detailed message + CHANGELOG entry, push to main.

A template is "hardened" when all six phases are green and the internal self-check passes.

## What's NOT yet done

- **End-to-end smoke test of Template 01.** Requires a Vapi or Retell test account. The maintainer has the account provisioning on the next sprint. The Memory pattern is proven via Template 02 (executions 445 + 446 both green) which uses the same backend, but the Vapi/Retell webhook ingestion path itself has not been smoke-tested live against a real call. This is the gating item for distribution push.
- **End-to-end smoke tests of T04-T08.** Each has been live-imported into n8n.studiomeyer.io v2.15.0 and pre-activation-check passed (all node types recognized, all connections valid). End-to-end live triggers against real backends (real Telegram bot for T04, real web-chat widget for T05, real Pipedrive instance for T06, real Fathom or Otter for T07, real Mem0 or Zep account for T08) are part of the next sprint when those provisioning items land.
- **`n8n-nodes-base.openAi` vs `@n8n/n8n-nodes-langchain.openAi` decision.** All eight templates currently use `n8n-nodes-base.openAi` (verified working in n8n 2.15.0 pre-activation check). Newer n8n versions also expose `@n8n/n8n-nodes-langchain.openAi`. Templates may migrate to the LangChain variant in v0.6 once we verify it works across the n8n version matrix.
- **Pagination beyond first page in T08.** The migration template fetches a single paginated call from Mem0 or Zep. For source datasets > 1000 records, wrap `Fetch from Source` in a Loop with cursor-tracking. Documented in the README, not implemented in the workflow yet.
- **Distribution push** to n8n.io/workflows, awesome-n8n-templates PR, dev.to long-form articles, Reddit r/n8n + r/AI_Agents, LinkedIn DACH PDF carousels. Held until Template 01 end-to-end smoke is documented. Plan: T08 ships first to Hacker News Show HN as the migration whitespace is wide open in 2026.

## Public claims discipline

A repo-internal rule: **public repo description never claims more than the weakest production-ready path in the repo claims.**

GitHub repo description: "Production-pattern hardened n8n workflow templates with cross-session memory."
Top-level README: "Production-pattern hardened, not a one-click production deploy yet" with explicit per-template status.
This `STATUS.md`: ground truth, updated per release.

If you find a discrepancy between any of these, file an issue. The README and STATUS.md are the source of truth, the GitHub description tracks them.

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io).*
