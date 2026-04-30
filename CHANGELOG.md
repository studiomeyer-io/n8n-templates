# Changelog

All notable changes to this repository will be documented here. The format is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
