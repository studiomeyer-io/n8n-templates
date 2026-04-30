# Changelog

All notable changes to this repository will be documented here. The format is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.3.0] - 2026-04-30

### Added

- **Production patterns block** in every template README. Five sections covering idempotency, error branches with the correct `{{ $error.message }}` syntax (the deprecated `$json.execution.error.message` did not work), webhook HMAC verification (Vapi `x-vapi-signature`, Retell `x-retell-signature`, Telegram secret-token, Stripe), rate limiting (60 requests / 5 min / IP, configurable), and memory de-duplication via the gatekeeper. These are the patterns that distinguish a "demo this" template from a "ship this" template, and they are missing from every other public n8n template repo we audited.
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
