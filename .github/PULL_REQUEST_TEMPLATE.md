# Pull request

## What this PR does

One paragraph. New template? Fix in existing template? Repo-wide infrastructure change?

## Brand Bibel check

Confirm the change passes the [N8N-BRAND-BIBEL.md](../N8N-BRAND-BIBEL.md) self-check:

- [ ] All 12 README pflicht-sections present (if README touched)
- [ ] No em-dashes (the `U+2014` character) in any markdown or workflow.json. CI runs the validate-workflows GitHub Action which fails on any em-dash match in `templates/`.
- [ ] `workflow.json` validates (Missing refs: NONE)
- [ ] No real credentials, secrets, or test pinData committed
- [ ] All `>> SET ME <<` markers visible as Sticky Notes
- [ ] Multi-Provider pattern in place (if LLM call exists)
- [ ] Cover image generated and committed (`cover.png`)
- [ ] Smoke-tested in a real n8n instance (paste execution-id below)
- [ ] Cross-links to related templates work
- [ ] Top-level CHANGELOG.md updated

## Smoke-test evidence

n8n execution-id: `<paste>`
Backend hit: `memory.studiomeyer.io` / other: `<paste>`

## Screenshots / GIFs

If the change is visual (cover image, sticky-note layout), include before/after screenshots.

## Related issues

Closes #
References #
