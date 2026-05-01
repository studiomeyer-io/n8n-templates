# Cover Image Spec, Mem0 / Zep Migration to StudioMeyer Memory

## Output

- **File:** `cover.png` (this folder, sibling of workflow.json + README)
- **Size:** 1216x640 (auto-cropped to 1200x630 by n8n.io and Twitter)
- **Style:** Match the rest of the n8n-templates suite. Dark navy background, gold accent for the destination memory layer, clean technical illustration.

## Flux prompt

```
A clean technical illustration on a deep navy background. Three icons in a horizontal row connected by glowing arrows: (1) two stacked database-cylinder icons on the left in soft blue labeled "Mem0" and "Zep" (small text underneath each), (2) a flowing-data-arrow with batch-records visible in soft white center, (3) a knowledge-graph node cluster in gold on the right showing entities with branched observations, the destination. Below the icons in bold sans-serif white text: "Migrate to StudioMeyer Memory". Below that in smaller gold text: "n8n + StudioMeyer Memory". Minimalist, high contrast, no clutter, suitable for a developer marketplace listing.
```

## Generation

```
mcp-media generate_image with provider=flux model=flux-pro-v1.1, size=1216x640
```

## Status checklist

- [ ] Image generated
- [ ] Saved as cover.png in this folder
- [ ] Used in n8n.io/workflows submission
- [ ] Used in dev.to article header
- [ ] Used in LinkedIn carousel slide 1
- [ ] Used in Reddit + Hacker News post (linked, not inline since auto-pulls preview)
