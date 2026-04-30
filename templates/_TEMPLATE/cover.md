# Cover Image Spec — `<Template Title>`

## Output

- **File:** `cover.png` (this folder, sibling of workflow.json + README)
- **Size:** 1200x630 (Twitter Card + n8n.io + dev.to compatible)
- **Style:** Match the rest of the n8n-templates suite. Dark navy background, gold accent for the memory layer, clean technical illustration, no marketing clutter.

## Flux prompt

Replace `<TRIGGER ICON>`, `<LLM ICON>`, `<TEMPLATE TITLE>` with the specifics of this template. Keep the rest of the prompt identical so all covers in the suite look like a family.

```
A clean technical illustration on a deep navy background. Three icons in a horizontal row connected by glowing arrows: (1) <TRIGGER ICON> on the left in soft blue or white, (2) a knowledge-graph node cluster in gold center, (3) <LLM ICON> on the right in soft cyan. Below the icons in bold sans-serif white text: "<TEMPLATE TITLE>". Below that in smaller gold text: "n8n + StudioMeyer Memory". Minimalist, high contrast, no clutter, suitable for a developer marketplace listing.
```

## Generation

```
mcp-media generate_image with provider=flux model=flux-pro-v1.1, size=1216x640
```

(n8n.io auto-crops 1216x640 to 1200x630.)

## Status checklist

- [ ] Image generated
- [ ] Saved as `cover.png` in this folder
- [ ] Used in n8n.io/workflows submission
- [ ] Used in dev.to article header
- [ ] Used in LinkedIn carousel slide 1
- [ ] Used in Reddit post (linked, not inline since Reddit auto-pulls preview)
