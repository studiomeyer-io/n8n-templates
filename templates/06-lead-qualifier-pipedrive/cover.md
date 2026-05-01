# Cover Image Spec, Lead-Qualifier with BANT+I and Pipedrive

## Output

- **File:** `cover.png` (this folder, sibling of workflow.json + README)
- **Size:** 1216x640 (auto-cropped to 1200x630 by n8n.io and Twitter)
- **Style:** Match the rest of the n8n-templates suite. Dark navy background, gold accent for the memory layer, clean technical illustration.

## Flux prompt

```
A clean technical illustration on a deep navy background. Three icons in a horizontal row connected by glowing arrows: (1) a clipboard-form-with-checkmarks icon on the left in soft blue representing the inbound form, (2) a knowledge-graph node cluster in gold center showing a lead entity with branched BANT-score observations, (3) a CRM-pipeline-funnel icon on the right in soft cyan with stages hot/warm/cold visible. Below the icons in bold sans-serif white text: "Lead-Qualifier with BANT+I and Pipedrive". Below that in smaller gold text: "n8n + StudioMeyer Memory". Minimalist, high contrast, no clutter, suitable for a developer marketplace listing.
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
- [ ] Used in Reddit post (linked, not inline since Reddit auto-pulls preview)
