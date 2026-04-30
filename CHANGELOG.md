# Changelog

All notable changes to this repository will be documented here. The format is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- **Template 02 hardened to v0.4.0 production-pattern standard** (first template to ship the patterns as actual opt-in nodes in workflow.json instead of documentation-only). Three new opt-in Code nodes between trigger and customer-key extraction: `Verify Webhook (opt-in)` (gated by `WEBHOOK_INTEGRITY_CHECK_ENABLED=1`), `Rate Limit (opt-in)` (gated by `RATE_LIMIT_ENABLED=1`), `Idempotency Check (opt-in)` (gated by `IDEMPOTENCY_ENABLED=1`). Each pass-through when its env var is unset, so the default import still boots clean. Production deployments toggle all three plus the Telegram Trigger `secretToken`. Concurrency disclaimer for `$getWorkflowStaticData` documented inline plus production-recommendation snippets for Redis and reverse-proxy migration paths.
- **Error-output branch wired in workflow.json** (Template 02). Both `OpenAI Reply` and `Anthropic Reply` have `onError: continueErrorOutput` enabled. The error pin lands at the new `LLM Fallback Reply` Code node, which builds a graceful customer-facing message and feeds two destinations: `Telegram Reply` (so the customer gets an answer instead of silence) and a new `Memory: Learn Error` write with `category: mistake, tags: [llm-error, <provider>]`. The `Route by Provider` switch's fallback output (typo or unknown provider value) also lands at `LLM Fallback Reply`, so a misconfigured `provider` field still produces a reply.
- **Live-verification table** in Template 02 README. Two real n8n executions documented: 445 (new-customer-create branch) and 446 (known-customer-open branch), both green against memory.studiomeyer.io with smoke-test execution proof.
- **"How this compares" comparison table** in Template 02 README. Eight-row capability matrix vs Mem0, Zep, and Memori with concrete free-tier numbers (10K memories + 1K retrieval/mo for Mem0, 1k credits/mo + Graphiti OSS for Zep) and bi-temporal / EU-hosting / OAuth coverage.
- **Multi-provider switch section** as standalone block in Template 02 README. Step-by-step instructions for switching OpenAI to Anthropic and for adding a third provider (Gemini, Mistral, Ollama) including the on-error wiring and Normalize-output extension pattern.
- **Customer health score** as fifth Extending pattern in Template 02 README.
- **Hard compatibility floor section** as standalone block in Template 02 README (matching brand-bibel section 10c).

### Fixed

- **Anthropic node type-string** in Template 02 workflow.json. Was `n8n-nodes-base.anthropic` (does not exist in n8n core, produces "Unrecognized node type" on activation). Now `@n8n/n8n-nodes-langchain.anthropic` with `resource: text, operation: message` parameters. Verified against n8n 2.15.0 pre-activation check. The same bug exists in Templates 01 and 03 and should be fixed in their next pass.
- **Anthropic model in Template 02 workflow.json** corrected from `claude-sonnet-4-6` to `claude-haiku-4-5`. The README and brand-bibel always said Haiku 4.5, the workflow.json was a leftover from the v0.1 spec. Setup section also lost a stray "this template uses Sonnet 4.6 for richer replies" line. Haiku 4.5 is the correct choice for support-bot reply latency and cost.
- **Customer-key extraction in Template 02** no longer produces `tg:undefined` for channel posts and forwarded messages without a sender. New fallback chain: email > Telegram user id > Telegram chat id, with a hard `throw` when all three are missing. Closes a silent collision bug where every anonymous sender accumulated into one shared customer entity.
- **`Route by Provider` fallback output in Template 02 workflow.json** is now wired to `LLM Fallback Reply` instead of dead-ending. A typo or unknown `provider` value (e.g. `"groq"` instead of `"openai"`) used to silently drop the workflow item. Now it produces a graceful fallback reply and a `category: mistake` learning.
- **OpenAI gpt-5-mini pricing in Template 02 README** corrected from $0.15 / $0.60 per 1M to $0.25 / $2.00 per 1M (current pricing as of 2026 verified via pricepertoken.com and openai.com/api/pricing). Per-ticket cost recalculated to ~$0.0014 (was understated as ~$0.0004). Worked example at 5 000 tickets/mo: ~EUR 36/mo (was ~EUR 31/mo).
- **Brand-bibel section order in Template 02 README** restored. Common gotchas now precedes Production Patterns (sections 10 then 10b), Hard Compatibility Floor follows Production Patterns (10c), Tech Stack Matrix and Credentials Checklist follow (10d, 10e). Live verification and How this compares retain their extra-section status after Credentials Checklist.
- **Sticky-note colors in Template 02 workflow.json** changed from color 4 (not in brand-bibel palette) to color 7 (explanation orange) on the Production Patterns and Error Branch sticky notes.

### Notes

- This is the v0.4.0 prep release. Template 02 is the first template to land the four production patterns as wired-in nodes. Template 01 (Voice Agent) and Template 03 (Personal Assistant) follow in the next two passes, each carrying the same Anthropic type-string fix and the same opt-in node pattern. Distribution push (n8n.io/workflows, awesome-n8n-templates, dev.to, Reddit, LinkedIn) holds until all three templates ship at v0.4.0 standard.
- Smoke-test against memory.studiomeyer.io live: executions 445 + 446 both green, both branches of the Known Customer IF exercised, full Memory pattern (Lookup + Open or Create + Observe + Learn) confirmed against the production backend.
- Three dedicated agents reviewed the template before commit (Critic + Architect + Research). Two AMBER + one AMBER. Eight findings total, seven fixed before commit, one (Research's claim that `n8n-nodes-base.openAi` is unrecognized in current n8n) rejected after live verification showed n8n 2.15.0 recognizes the type and only complains about missing credentials.

## [0.3.1] - 2026-04-30

### Fixed

Response to the second 3-agent code review pass (analyst + critic + research, all three completed this time).

- **Honesty fix in every template README and the top-level README.** v0.3.0 said "These five patterns ship in the workflow.json". They don't. They are documented as drop-in code-node snippets in `N8N-BRAND-BIBEL.md`. The current workflow.json ships the happy path. The patterns are opt-in additions you wire in for hardened production. Fixed every template README to say so explicitly. The roadmap entry for v0.4.0 commits to wiring the four production-patterns as opt-in nodes inside the default workflow.json so they're one click away rather than one paste away.
- **Error-handling syntax (Critic v2 P1.1).** The previous claim "use `{{ $error.message }}`" was flat wrong. `$error` does not exist as an n8n expression context. Fixed to `{{ $json.error.message }}` for the inline error pin (downstream of "Continue (using error output)") and noted that `{{ $json.execution.error.message }}` is the separate, also-valid syntax for the Error Trigger Workflow (Workflow Settings → Error Workflow). Both are documented in the n8n Docs. The wrong claim appeared in eight places: brand bibel, top-level README, changelog, and three template READMEs. All eight fixed and verified.
- **HMAC `timingSafeEqual` length-guard (Critic v2 P1.4).** `crypto.timingSafeEqual` throws `RangeError` if the two buffers differ in length, which an attacker can trigger with a one-character signature, turning the workflow into a free DoS vector. Brand bibel HMAC snippet now does an explicit `if (sigBuf.length !== expBuf.length) throw new Error('HMAC verification failed')` before the timing-safe compare. Snippet also covers Stripe's `t=<timestamp>,v1=<hmac>` format with the recommendation to use `stripe.webhooks.constructEvent` from the SDK rather than re-implementing the parsing.
- **Idempotency disclaimer (Critic v2 P1.2).** Brand bibel idempotency snippet now spells out that `$getWorkflowStaticData('global')` is not atomic and not cluster-aware. Two concurrent executions with the same key can both pass the dedup check in the millisecond window between read and write. Two n8n workers behind a load balancer don't share the static-data map at all. Default in-memory pattern is good enough for single-instance dev / small production loads. Production-empfehlung: Redis SET NX with EX 300 NX (atomic, cluster-aware, cleans up via TTL). Snippet provided.
- **Rate-limit disclaimer (Critic v2 P1.3).** Brand bibel rate-limit snippet expanded with three improvements: (a) explicit TOCTOU note (we may overshoot the limit by ~5% under concurrent fires), (b) `MAX_BUCKETS = 5000` cap on the in-memory map with eviction of expired entries to prevent unbounded memory growth, (c) production recommendation to handle rate-limiting at the reverse proxy (Nginx `limit_req_zone`, Cloudflare WAF, AWS WAF, Traefik `RateLimit` middleware) for atomic + cluster-aware + faster behavior. The Code-Node pattern remains as defense-in-depth or for setups without a reverse proxy.
- **Em-dashes in cover.md files (Analyst v2 finding).** Four cover.md files (templates 01/02/03 + _TEMPLATE) had one em-dash each that the v0.2.0 sweep missed. Cleaned. CI em-dash guard in `.github/workflows/validate-workflows.yml` now covers cover.md too.
- **`meta` field in workflow.json (Analyst v2 finding).** The brand bibel forbids `meta` because the n8n Public API rejects it on POST `/api/v1/workflows`. v0.2.0 templates still had `meta: { templateId: "..." }` left over. Stripped from all three templates. The validate-workflows CI now blocks `meta` field presence on PR.

### Added

- **"How we compare to other public n8n template repos" section** in the top-level README. Thirteen-row capability matrix that makes the production-patterns gap visible at first scroll. Driven by the research-agent finding that no competitor ships these patterns.
- **FAQ section** in the top-level README with eight questions covering memory dependency, multi-provider rationale, n8n version floor, Cloud compatibility, cost-per-execution, production-readiness honesty, contribution flow, sticky-note verbosity, and security-issue reporting.
- **Distribution status table** in the top-level README. Explicit list of which channels we are on (GitHub repo, topics, social preview, discussions) and which we hold back until v0.4.0 ships the production patterns as wired-in nodes (n8n.io/workflows, awesome-n8n-templates, dev.to, Reddit, LinkedIn). Honesty about the maturity gap up front.

### Notes

- v0.3.1 is a patch release because no template's behavior changed, only documentation and brand-bibel snippets. v0.4.0 will be the next minor and will move the four opt-in production patterns from documented snippets into actual nodes inside the default workflow.json (off by default with sticky-note enable instructions). That is the version we submit to n8n.io and awesome-n8n-templates.

## [0.3.0] - 2026-04-30

### Added

- **Production patterns block** in every template README. Five sections covering idempotency, error branches with the correct `{{ $json.error.message }}` syntax for inline error pins (`{{ $json.execution.error.message }}` is for the separate Error Trigger Workflow, both are documented n8n syntaxes), webhook HMAC verification (Vapi `x-vapi-signature`, Retell `x-retell-signature`, Telegram secret-token, Stripe), rate limiting (60 requests / 5 min / IP, configurable), and memory de-duplication via the gatekeeper. These are the patterns that distinguish a "demo this" template from a "ship this" template, and they are missing from every other public n8n template repo we audited.
- **Tech stack matrix** in every template README. Concrete versions, costs, free-tier limits, and required-when columns for n8n, the community node, StudioMeyer Memory, OpenAI / Anthropic, and the trigger provider.
- **Credentials checklist** in every template README. Four-item checkbox list with where to get each key, which auth-mode to pick, and what test endpoint confirms it works.
- **Hard compatibility floor** of n8n 2.10.1 declared in the top-level README and brand bibel. Background: CVE-2026-27493 (CVSS 9.5, fixed Feb 2026) is an unauthenticated RCE in Form nodes. None of these templates use Form nodes, but no one should run a vulnerable n8n in any case. 1.x users: upgrade to 1.123.22 or later.
- **"What makes these templates different" section** in the top-level README. A six-row table listing the production patterns, why each one matters, and where in the workflow they live. Driven by the research-agent finding that no competitor template ships these patterns.

### Changed

- **N8N-BRAND-BIBEL.md** expanded with five new pflicht-sections covering Error Handling (correct n8n syntax), Idempotency (in-memory dedup with 5-min window, swap-to-Redis note), Webhook HMAC (HMAC-SHA256 + timing-safe-equal Code-Node skeleton), Rate Limiting (per-IP bucket pattern), and a 10b/c/d/e block for Production Patterns / Hard Compatibility Floor / Tech Stack Matrix / Credentials Checklist as mandatory README sections.
- The em-dash guard in `.github/workflows/validate-workflows.yml` already enforced the no-em-dash rule, but the rule is now also called out explicitly in the brand bibel under "Voice-Regeln".

### Notes

- This release is the response to the 3-agent code review (analyst, critic, research). Critic flagged CVE-2026-27493 awareness and a missing pre-release validator. Research found that the production patterns above are the differentiator that no competitor template repo ships. Analyst hung on the first `agent_recall` tool call (the repo is markdown + JSON, not source code, so codebase-memory-mcp had no symbols to bind), so we cannot include analyst findings in this release.

## [0.2.0] - 2026-04-30

### Added

- **N8N-BRAND-BIBEL.md** as the single source of truth for tone, structure, branding, and quality gates across every template in this repo.
- **Multi-provider switch** in template 01 (Voice Agent Cross-Session Memory). A `Set Provider` + `Route by Provider` pair lets builders choose OpenAI (default) or Anthropic, or extend with a third branch (Gemini, Mistral, local Ollama). Both branches converge in `Normalize LLM Output` so memory writes stay identical regardless of provider.
- **`.github/` standard scaffolding**: FUNDING.yml, ISSUE_TEMPLATE (bug + template-request + config), PULL_REQUEST_TEMPLATE, and a validate-workflows GitHub Action that catches missing refs, real credentials, and em-dashes before merge.
- **`templates/_TEMPLATE/` skeleton** for new templates so every contribution starts at the brand-bibel baseline.
- **CHANGELOG.md, CODE_OF_CONDUCT.md, SECURITY.md, ECOSYSTEM.md** as standard repo top-level files.

### Changed

- **Default OpenAI model** in template 01 from `gpt-4o-mini` (deprecated) to `gpt-5-mini`. Anthropic stays on `claude-haiku-4-5` for the speed-tier branch.
- **Template 01 README** rewritten to reflect the new multi-provider pattern, with a clearer architecture diagram and explicit instructions for adding additional LLM branches.

### Fixed

- Validation guard in CI rejects PRs where `workflow.json` has missing node references, embedded test pinData, or em-dashes in markdown / workflow files.

## [0.1.0] - 2026-04-30

### Added

- **Three Tier-1 templates** ready to import into any n8n instance:
  - `01-voice-agent-cross-session-memory` (Vapi/Retell webhook, caller recognition, post-call memory writes)
  - `02-customer-support-with-history` (Telegram trigger, email-based customer entity, dossier injection into Claude prompt)
  - `03-personal-assistant-long-term-memory` (Telegram intent classifier, note / ask / summary modes)
- **Per-template README** with architecture diagram, memory model, setup steps, extension ideas, cost notes, and gotchas.
- **Per-template `cover.md`** with Flux generation prompt for the 1200x630 cover image (n8n.io / dev.to / LinkedIn carousel).
- **CONTRIBUTING.md** with the contribution structure and review checklist.
- **MIT LICENSE.**

[Unreleased]: https://github.com/studiomeyer-io/n8n-templates/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/studiomeyer-io/n8n-templates/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/studiomeyer-io/n8n-templates/releases/tag/v0.1.0
