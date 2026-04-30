# N8N Brand Bibel

> **Wer dieses File liest:** Jeder (Mensch oder LLM) der ein neues n8n-Template fuer dieses Repo schreibt, ein bestehendes ueberarbeitet, oder den Distributions-Push fuer ein Template plant. Dieses File ist die Single Source of Truth fuer Tonalitaet, Struktur, Branding und Quality-Gate. Wenn etwas in diesem File widerspricht zu CONTRIBUTING.md oder einem einzelnen Template-README, gewinnt dieses File.

---

## Mission in einem Satz

Jedes Template in diesem Repo ist ein **drop-in-Asset** das ein Builder in seinem n8n-Stack innerhalb von 5 Minuten produktiv betreiben kann, mit oder ohne StudioMeyer Memory, mit oder ohne LLM-Provider-Lock-in, ohne dass er sich durch Marketing-Geschwafel oder unklaren Code arbeiten muss.

## Die drei Standard-Tests

Jedes Template muss vor Commit alle drei passen:

1. **5-Minuten-Test.** Ein durchschnittlicher n8n-User klont das Repo, importiert die `workflow.json`, fuellt die `>> SET ME <<` Marker, klickt Activate, hat das Ding produktiv. Ohne Editor-Hacking.
2. **Read-the-README-Test.** Wenn der User nur das README liest und die `workflow.json` nie sieht, versteht er trotzdem was passiert, welche Side-Effects es gibt, und wie er erweitert.
3. **Cold-Search-Test.** Ein Builder findet das Template ueber eine Google-Suche wie "n8n voice agent memory" oder "n8n customer support claude" und sieht beim ersten Scroll dass es genau seine Frage loest.

Wenn auch nur einer der drei nicht passt, ist das Template nicht ready.

---

## Tonalitaet

**Was wir sind.** Ein Studio aus Mallorca das seit 2024 MCP-Server, AI-Agents und n8n-Workflows gebaut hat. Wir sprechen direkt, technisch dicht, ohne Marketing-Floskeln. Wir lassen ehrlich, wenn etwas nicht funktioniert. Wir nennen Limitations bevor sie der Reviewer findet.

**Was wir nicht sind.** Ein Marketing-Team. Ein generic SaaS-Anbieter. Ein "AI Solutions Partner". Wir verkaufen nicht, wir liefern Code der lebt, dazu honest README's.

### Voice-Regeln (zwingend)

- **Echte Saetze.** Keine Bullet-Listen mit fett-gedruckten Headern wenn ein Satz reicht. Bullets sind erlaubt, aber nicht als Mittel um nicht denken zu muessen.
- **Erste Person Plural** ("we built this so...") oder neutrale technische Sprache. Nie "you should" oder "users typically".
- **Konkret statt abstrakt.** Statt "fast" → "under 25ms latency". Statt "production-ready" → "tested with 1000 calls/day in our own setup".
- **Keine Em-Dashes** (das lange `U+2014` Zeichen). Stattdessen Komma, Punkt oder Klammer. Em-Dashes gelten 2026 als KI-Signatur und werden in Indexing-Algorithmen abgewertet.
- **Keine "Hope this helps" / "Let me know if..." Floskeln** am Ende von Sektionen.
- **Englisch fuer alle README's, Codes-Comments, Commit-Messages.** Internes Werkstatt-Material (dieses File hier, CONTRIBUTING.md interne Sektionen) darf Deutsch sein wenn das schneller geht.
- **Imperfekt erlaubt.** Ein angefangener Gedanke der weitergeht ist menschlicher als ein perfekter Marketing-Satz.

### Was IMMER weglassen

- Em-Dashes
- "Hope this helps"
- "Excited to share"
- "We're thrilled"
- "Empowering builders to..."
- "Unlock the power of..."
- TL;DR Blocks am Anfang oder Ende
- Doppelte Anfuehrungszeichen fuer Hervorhebung wenn kein echtes Zitat
- Drei-Satz-Trigger ("First X. Second Y. Third Z.")
- Bullet-with-bold-Header Strukturen die nicht inhaltlich erforderlich sind

---

## README-Struktur (Pflicht)

Jeder Template-README MUSS diese Sektionen in dieser Reihenfolge haben. Sektionen weglassen oder umsortieren = nicht ready.

### 1. StudioMeyer-MCP-Stack-Banner (oberste Zeile)

```markdown
<!-- studiomeyer-mcp-stack-banner:start -->
> **Part of the [StudioMeyer MCP Stack](https://studiomeyer.io)**, Built in Mallorca · ⭐ if you use it
<!-- studiomeyer-mcp-stack-banner:end -->
```

### 2. Title + Tagline + Cover

```markdown
# Template Title (Multi-Provider)

> One-sentence value prop. What does the user get?

![Cover](./cover.png)
```

### 3. What this does

Zwei kurze Absaetze. Erste = Daten-Fluss in einer Reihe ("Voice provider sends webhook → workflow looks up caller in Memory → builds context-aware prompt → LLM replies → outcome persisted"). Zweite = Kill-Phrase fuer den Suchenden ("The result is ein voice agent that knows your customer the second time they call").

### 4. Architecture (ASCII-Diagramm)

Vertikales Diagramm mit unicode-Box-Drawing-Chars (`│`, `▼`, `┌`, `└`, `┐`, `┘`, `┴`, `┬`, `─`, `▼`, `►`). Stil:

```
[Webhook]
    │
    ▼
[Code: Normalize]              ← parses Vapi or Retell payload
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

Erklaerungs-Pfeile rechts (`← parses ...`) wenn der Node-Name selbst nicht klar genug ist.

### 5. Memory Model (Tabelle)

```markdown
| Concept | Storage | Key |
|---|---|---|
| Caller identity | `Entity` of `entityType: caller` | normalized phone (E.164) |
| Per-call detail | `Observation` on the caller entity | timestamp + transcript snippet |
| High-level outcome | `Learning` (category: insight) | tagged with `voice-agent` |
```

Wenn das Template kein Memory nutzt, diese Sektion durch **State Model** ersetzen mit aequivalenter Tabelle (was wird wo persistiert).

### 6. Setup (numbered list)

Maximal 7 Schritte. Wenn mehr nötig, Template ist zu komplex und sollte gesplittet werden.

```markdown
1. **Install community node** (one-liner cmd)
2. **Get API key** (with link)
3. **Add credentials** (welche, von wo)
4. **Import workflow** (Schritt im n8n)
5. **Configure** (welche Set-Me-Marker)
6. **Wire to provider** (Vapi/Retell Setup-Snippet)
7. **Test** (was hits man als erstes)
```

### 7. Multi-Provider Switch (wenn LLM-Call drin ist)

Erklaert wie `Set Provider` + `Route by Provider` funktionieren. Wie der User einen dritten Provider (Gemini, Mistral, local Ollama) als Branch dazubaut. Vier Schritte.

### 8. Extending (3-4 konkrete Ideen)

Pro Idee ein Absatz, kein Bullet-Header. Pattern: **Idee** in fett am Anfang des Absatzes, dann der Plan in Fliesstext.

```markdown
**Add caller scoring.** Nach `Memory: Lookup Caller`, branch on observation count. Caller mit 5+ prior interactions get a different greeting. Use `entity.open` to fetch the full observation list.

**Hand off to humans.** Add an IF node after Claude that checks for sentiment keywords ("frustrated", "speak to a manager"). Branch into a Slack notification with the caller's history pre-formatted.
```

### 9. Cost notes (Tabelle)

```markdown
| Component | Approx cost |
|---|---|
| 2× Memory operations | < $0.001 |
| 1× LLM call (~500 tokens) | ~$0.0005 |
| **Total per execution** | **~$0.002** |
```

Plus ein Satz mit Schaetzung pro Monat bei realistic Volume.

### 10. Common gotchas (3-5 Stueck)

Pro Gotcha: ein Bold-Lead-Phrase als Bullet-Start, dann Erklaerung in Fliesstext. Was geht schief, warum, wie behebt man es.

### 10b. Production Patterns (NEU, Pflicht ab v0.3)

Ein Block der in JEDEM Template README zwischen "Common gotchas" und "Related templates" steht. Vier Sub-Sections:

**Idempotency.** Welcher Field eindeutig identifiziert, dass ein Trigger-Event schon einmal verarbeitet wurde? Code-Snippet das ein in-memory-Set fuer 5-Minuten-Window haelt (Default) oder Redis SET NX Pattern (fuer Production-Skalierung). Pro Template:
- 01 Voice Agent: `callId` (Vapi `message.call.id` / Retell `call.call_id`)
- 02 Customer Support: Telegram `update_id` oder `message.message_id`
- 03 Personal Assistant: Telegram `update_id`

**Error Branches.** Jeder LLM-Call und jeder externe API-Call muss einen Error-Output haben der nicht still failed. n8n-Syntax fuer Error-Daten ist `{{ $error.message }}` (NICHT `$json.execution.error.message`, das ist deprecated/falsch). AI-Agent-Nodes muessen auf "Stop and Return Error" gesetzt sein. Sticky Note dokumentiert das pro Template.

**Webhook Security.** Wenn der Trigger ein Webhook ist und der Anbieter HMAC-Signing unterstuetzt (Vapi via `signing-secret` Header, Retell via `X-Retell-Signature`, Stripe `Stripe-Signature`, Telegram via `X-Telegram-Bot-Api-Secret-Token`), muss ein Code-Node am Anfang die Signatur verifizieren und unsignierte Requests verwerfen. Code-Snippet als Inline-Sticky Note. Default-Implementation: HMAC-SHA256 mit timing-safe-equal.

**Rate Limiting.** Memory + LLM Calls sind nicht kostenlos. Wenn der Trigger ein Public-Webhook ist (also ohne Auth), muss eine Rate-Limit-Stage davor (n8n hat keinen native Rate-Limit-Node, also Code-Node mit IP-Map + Window) dafuer sorgen dass ein Angreifer den Bill nicht auf $1000 pumpt. Pro Template Default: 60 requests / 5 min / IP, ueberschreibbar.

### 10c. Hard Compatibility Floor (NEU, Pflicht ab v0.3)

Pro Template README in der Setup-Sektion explizit:

```markdown
**Minimum n8n version: 2.10.1** (CVE-2026-27493 fix). Older versions of n8n have an unauthenticated RCE vulnerability in Form nodes. This template does not use Form nodes itself, but you should still upgrade for general security. Older 1.x users: upgrade to 1.123.22 or later.
```

Plus im Top-Level README ein Banner.

### 10d. Tech-Stack-Matrix (NEU, Pflicht ab v0.3)

Tabelle in jedem Template README zwischen "Setup" und "Multi-Provider Switch". Was an Tools / Versionen / Tiers / Cost gebraucht wird. Beispiel:

```markdown
## Tech stack matrix

| Component | Version | Cost | Free tier | Required |
|---|---|---|---|---|
| n8n | >= 2.10.1 | self-hosted free / Cloud $20/mo | n/a | yes |
| n8n-nodes-studiomeyer-memory | >= 0.1.0 | free | n/a | yes |
| StudioMeyer Memory | API key | EUR 0 / 29 / 49 | 10k ops/month | yes |
| OpenAI (default) | gpt-5-mini | $0.15/1M input | $5 free credit | yes if provider=openai |
| Anthropic | claude-haiku-4-5 | $1/1M input | $5 free credit | yes if provider=anthropic |
| Vapi or Retell | varies | $0.07-0.15/min | trial credits | yes |
```

### 10e. Credentials Checklist (NEU, Pflicht ab v0.3)

Direkt unter dem Tech-Stack-Matrix. Jede Credential die der User in n8n anlegen muss, mit (a) wo bekomme ich sie her, (b) welcher Auth-Mode, (c) welcher Test-Endpoint zeigt dass es funktioniert.

```markdown
## Credentials checklist

Before activation, create these credentials in n8n:

- [ ] **StudioMeyer Memory API** (`studioMeyerMemoryApi`). Get key at memory.studiomeyer.io/dashboard/keys. Test: any memory.search call returns 200.
- [ ] **OpenAI API** (`openAiApi`) OR **Anthropic API** (`anthropicApi`). Get key at platform.openai.com / console.anthropic.com. Test: model list endpoint returns models.
- [ ] **Vapi / Retell** (provider-specific). Setup webhook URL in their dashboard pointing at your n8n instance.
- [ ] **HMAC signing secret** (optional but recommended). For provider HMAC verification. Get from Vapi/Retell webhook settings.
```

### 11. Related templates (Cross-Link Footer)

```markdown
## Related templates

- [02 - AI Customer Support with History](../02-customer-support-with-history/), same memory pattern for chat
- [03 - Personal Assistant with Long-Term Memory](../03-personal-assistant-long-term-memory/), single-user variant with tool use
```

### 12. StudioMeyer-Footer

```markdown
---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Part of the [StudioMeyer MCP Stack](./README.md#related). Memory at [memory.studiomeyer.io](https://memory.studiomeyer.io). Issues / questions at [github.com/studiomeyer-io/n8n-templates/issues](https://github.com/studiomeyer-io/n8n-templates/issues).*
```

---

## Workflow.json Standard

### Pflicht-Felder

- `name`: Template-Title (gleich wie README h1)
- `nodes`: array
- `connections`: object
- `settings`: `{ "executionOrder": "v1" }`
- `pinData`: `{}` (immer leer beim Commit)
- `meta`: weglassen (n8n Public API lehnt sonst ab)
- `active`: weglassen oder `false`
- `staticData`: weglassen

### Pflicht beim Commit

- Stable Node-IDs (`voice-1-webhook` statt UUID). n8n regeneriert bei Import sowieso.
- Sticky Notes als orange (color 5 fuer SET-ME / 6 fuer Intro / 7 fuer Erklaerung).
- Keine echten Credentials. `credentials`-Block leer lassen, n8n zeigt rote Marker im Editor und der User legt sie selbst an.
- Keine PinData (Test-Daten). Strippen vor commit.

### Multi-Provider Pattern (Pflicht wenn LLM-Call drin)

```
Build Prompt → Set Provider → Route by Provider ─┬─ OpenAI Reply ──┐
                                                  └─ Anthropic Reply ┘
                                                       ↓
                                                Normalize LLM Output
                                                       ↓
                                                Respond + Memory writes
```

Default-Provider ist `openai` (groessere Audience). User kann mit einem Klick auf `anthropic` umstellen. Code-Node `Normalize LLM Output` extrahiert `replyText` aus beiden Response-Shapes.

### Error-Handling Pattern (Pflicht wenn LLM-Call oder externer API-Call)

In n8n setzt man pro Node unter den 3-dot-Menu "On Error" auf "Continue (Error Output)" oder "Stop and Return Error" je nach Use-Case. Der Error-Output-Pin ist im Editor rot.

Korrekte Syntax fuer Error-Daten in nachfolgenden Nodes:

```
{{ $error.message }}
{{ $error.name }}
{{ $error.description }}
```

NICHT `{{ $json.execution.error.message }}` (deprecated, gibt undefined). NICHT `{{ $error }}` ohne field (rendert das Object als string, unbrauchbar).

Pro Template mindestens ein Error-Branch der die LLM-Failure abfaengt:
- LLM Reply Node: "On Error" = "Continue (Error Output)"
- Error-Branch landet in einem Code-Node der ein Fallback-Reply baut ("Sorry, our system is briefly unavailable. We'll get back to you within an hour.") und Memory mit category="mistake" + tag="llm-error" loggt
- Memory-Logging im Error-Branch ist Pflicht damit wir Fehler-Patterns sehen

### Idempotency Pattern (Pflicht wenn der Trigger replay-faehig ist)

n8n's Default ist fire-and-forget: bei einem Provider-Retry (Vapi sendet das gleiche `call_ended` zweimal, Telegram retryt bei Webhook-5xx) executet der Workflow zweimal und schreibt zweimal Memory.

Loesung: Code-Node am Anfang nach Trigger der einen Idempotency-Key extrahiert (Vapi `callId`, Telegram `update_id`, etc.) und gegen einen In-Memory-Set checkt. Wenn schon gesehen → return, sonst weiter.

```js
// At top of "Normalize Payload" code node, after extracting idempotencyKey:
const seen = $getWorkflowStaticData('global').seenKeys ?? {};
const now = Date.now();
// Purge entries older than 5 minutes
for (const k of Object.keys(seen)) if (now - seen[k] > 300000) delete seen[k];
if (seen[idempotencyKey]) {
  return [{ json: { skipped: true, reason: 'duplicate', idempotencyKey } }];
}
seen[idempotencyKey] = now;
$getWorkflowStaticData('global').seenKeys = seen;
```

Production-Skalierung: ersetze `$getWorkflowStaticData` mit Redis SET NX (`SET key value EX 300 NX`). `$getWorkflowStaticData` ist per-Workflow-Instance, nicht cluster-aware.

### Webhook HMAC Verification (Pflicht wenn Trigger-Provider HMAC-Signing anbietet)

Wenn der Anbieter eine Signing-Secret + Header-Signatur unterstuetzt, MUSS der erste Code-Node nach dem Webhook-Trigger den HMAC verifizieren und unsignierte oder falsch signierte Requests verwerfen.

Provider-Header (Stand 2026-04):
- Vapi: `x-vapi-signature` (HMAC-SHA256 von Body)
- Retell: `x-retell-signature`
- Telegram: `x-telegram-bot-api-secret-token` (vergleicht mit hinterlegtem Token, kein HMAC)
- Stripe: `Stripe-Signature` (HMAC-SHA256 mit timestamp-prefix)

Code-Node am Anfang:

```js
const crypto = require('crypto');
const secret = $env('VAPI_SIGNING_SECRET'); // or hardcoded with credential mention
const sig = $request.headers['x-vapi-signature'];
const body = JSON.stringify($request.body);
const expected = crypto.createHmac('sha256', secret).update(body).digest('hex');
if (!sig || !crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expected))) {
  throw new Error('HMAC verification failed');
}
return [{ json: $request.body }];
```

In Sticky-Note dokumentiert: User muss `VAPI_SIGNING_SECRET` als n8n env var setzen, sonst ist der Workflow public-callable und der Bill-Schaden potentiell hoch. Default-Behavior ohne secret: Workflow funktioniert weiter aber loggt Warning ins Memory.

### Rate Limiting Pattern (Pflicht wenn Trigger-Webhook public ist)

Public Webhook + LLM-Call = potentielle Bill-Bombe wenn Angreifer 1000x in einer Minute hittet. Default-Schutz: Code-Node am Anfang mit IP-basiertem Limit.

```js
const ip = $request.headers['x-forwarded-for']?.split(',')[0]?.trim() ?? $request.headers['x-real-ip'] ?? 'unknown';
const buckets = $getWorkflowStaticData('global').rateBuckets ?? {};
const now = Date.now();
const WINDOW_MS = 5 * 60 * 1000; // 5 minutes
const LIMIT = 60; // 60 requests per IP per window

const bucket = buckets[ip] ?? { count: 0, windowStart: now };
if (now - bucket.windowStart > WINDOW_MS) { bucket.count = 0; bucket.windowStart = now; }
bucket.count++;
buckets[ip] = bucket;
$getWorkflowStaticData('global').rateBuckets = buckets;

if (bucket.count > LIMIT) {
  throw new Error(`Rate limit exceeded for ${ip}: ${bucket.count} requests in window`);
}
```

Production-Skalierung: Redis INCR + EXPIRE. Das Pattern ist gut genug fuer Single-Instance-Deployments + skalierbar bis ~10k requests/Stunde.

Aktuelle Model-IDs (Stand 2026-04-30, [Memory `0d27da68`](https://memory.studiomeyer.io)):

- OpenAI: `gpt-5-mini` (default), `gpt-5.4-mini` (newest, OpenAI empfehlt das fuer neue Workloads)
- Anthropic: `claude-haiku-4-5` (current Haiku-Tier), `claude-sonnet-4-6` fuer richere Antworten
- **NIEMALS** `gpt-4o-mini` oder `claude-3-haiku`, beide veraltet.

Wenn Du im April 2027+ liest: das hier ist Stand April 2026. Vor Hardcoding einer Model-ID immer `brave_web_search` machen + Memory checken.

### Validation vor Commit

```bash
node -e "
const fs=require('fs');
const w=JSON.parse(fs.readFileSync('templates/NN-name/workflow.json','utf8'));
const names=new Set(w.nodes.map(n=>n.name));
const missing=[];
for (const [from, ports] of Object.entries(w.connections)) {
  if (!names.has(from)) missing.push('FROM '+from);
  for (const port of ports.main || []) {
    for (const link of port || []) {
      if (link && !names.has(link.node)) missing.push('TO '+link.node);
    }
  }
}
console.log('Nodes:', w.nodes.length, '| Connections:', Object.keys(w.connections).length, '| Missing:', missing.length === 0 ? 'NONE' : missing.join(', '));
"
```

Erwarte: `Missing: NONE`. Sonst Template kaputt.

---

## Cover Image Standard

### Spec

- **Format:** PNG, 1200x630 pixels (Twitter Card + n8n.io Marketplace + dev.to Hero kompatibel)
- **Stil:** Dark navy background (#0a0e27 oder aequivalent), 3 Icons in horizontaler Reihe verbunden durch leuchtende Pfeile, gold-Akzent (#d4af37 oder aequivalent) fuer den mittleren Memory-Layer.
- **Text:** Bold sans-serif white. Title (1 Zeile) + Subline "n8n + StudioMeyer Memory" in gold.
- **Konsistenz:** Selbe Farbpalette + Layout-Logik fuer alle Templates damit der Repo visuell als Suite wirkt.

### Flux Prompt Template

Pro Template `cover.md` mit konkretem Prompt. Variabel sind nur die 3 Icons + der Titel. Prompt-Skeleton:

```
A clean technical illustration on a deep navy background. Three icons in a horizontal row connected by glowing arrows: (1) [TRIGGER ICON], (2) a knowledge-graph node cluster in gold center, (3) [LLM ICON]. Below the icons in bold sans-serif white text: "[TEMPLATE TITLE]". Below that in smaller gold text: "n8n + StudioMeyer Memory". Minimalist, high contrast, no clutter, suitable for a developer marketplace listing.
```

Generation:

```ts
mcp-media generate_image with provider=flux model=flux-pro-v1.1, size=1216x640
```

Output landet im Template-Folder als `cover.png`. cover.md dokumentiert den Prompt und Status (generiert / used in n8n.io / used in dev.to).

---

## Distribution-Standards

Nach jedem neuen Template oder grossen Update gilt fuer Distribution:

### GitHub-Repo

- Topics: `n8n`, `n8n-templates`, `ai-agents`, `voice-agents`, `mcp`, `workflow-automation`, plus templatespezifische
- Discussions enabled (Default: aus, muss aktiviert werden)
- About-Description ueberprueft
- Tag jeder substanziellen Aenderung mit Semver (`v0.1.0`, `v0.2.0`, etc.)

### n8n.io Marketplace Submit

- Cover-Image bereits da (Flux generated)
- Title gleich README h1
- Description: erste 2 Saetze des "What this does"
- Categories: AI, Bot, Webhook (template-spezifisch)
- Workflow-JSON direkt aus Repo

### dev.to Long-Form Tutorial

- 1500-2500 Worte
- Cover-Image gleich wie Repo
- Canonical URL: `studiomeyer.io/blog/n8n-{slug}`
- Tags: `#n8n`, `#ai`, `#javascript`, `#tutorial`
- Erscheint Di-Do 14-16 CET (US East Coast Morning)

### Reddit (r/n8n + r/AI_Agents)

- Format: Reddit-style locker, kein Marketing
- Hook: konkrete Zahl + Use-Case-Sprache ("Recognize callers across calls. 250 LOC in n8n.")
- Link erst im ersten eigenen Kommentar nach dem Post (Reddit-Algorithmus belohnt das)

### LinkedIn (DACH Audience)

- PDF-Carousel (7 Slides), kein Single-Image-Post (Single Images haben -30% Reach gegen Text-Posts)
- Slide 1 = Hook, Slides 2-5 = Wertversprechen, Slide 6-7 = CTA + GitHub-Link
- Posting-Zeit: Di-Do 07-09 CET

### awesome-n8n-templates PR

Sobald 5+ Templates LIVE: PR an [awesome-n8n-templates](https://github.com/restyler/awesome-n8n-templates) (280+ Stars Repo) mit Sektion `## StudioMeyer Memory Templates`. Maintained by uns, weil wir die Suite kontinuierlich ausbauen.

---

## Versions-Disziplin

- **Pro Repo-Update** ein CHANGELOG-Eintrag.
- **Versions-Schema:** Semver. PATCH (0.0.X) fuer Bug-Fixes in Templates, MINOR (0.X.0) fuer neue Templates oder Feature-Additionen, MAJOR (X.0.0) fuer Breaking-Changes (Tool-IDs, Node-Type-Aenderungen).
- **Tags pushen:** `git tag v0.1.0 && git push origin v0.1.0` nach jedem MINOR/MAJOR.

---

## Anti-Patterns die wir nicht zulassen

1. **Template ohne live n8n-Test.** Vor Commit muss das Template einmal in einer echten n8n-Instanz importiert + executed worden sein. Smoke-Test gegen `memory.studiomeyer.io` (oder das jeweilige Backend) mit echten Credentials. Nicht nur "JSON parses".
2. **Hardcoded Model-IDs ohne Stand-Datum.** Wenn ein Model-Name im Template steht, im Brand-Bibel-Kommentar oder der Memory-Decision das Datum dazu. Sonst sind wir in 6 Monaten outdated und Fragezeichen warum's bricht.
3. **Marketing-Sprache in technischer Doku.** "Empowering builders" / "Unlock the power" / "Game-changing" - wer das schreibt, schreibt's nochmal.
4. **5+ Sektionen ohne ein einziges Code-Snippet oder eine konkrete Zahl.** Wenn das README abstrakt bleibt, wer setzt das Template ein?
5. **Em-Dashes (, ) im README oder workflow.json.** LLM-Indexing erkennt das als KI-Signatur.
6. **Sticky Notes voller Text-Ueberlauf.** Sticky Notes sind kurze Marker, nicht Mini-Tutorials. Wenn der Hinweis 5 Saetze braucht, gehoert er ins README.
7. **Workflows ohne Sticky Notes.** Wer importiert sieht n8n-Editor-Boxen ohne Kontext. Mindestens ein Intro-Sticky am Anfang plus SET-ME-Marker bei jedem Punkt der User-Konfig.

---

## Quality-Gate vor jedem Push (3 Agents)

Nach jeder substanziellen Aenderung am Repo:

1. **Code-Review-Agent (Critic).** Sucht Bugs, Sicherheitsprobleme, Logikfehler in `workflow.json`. Pruft IF-Conditions, Variable-References, Race-Conditions zwischen Memory-Writes.
2. **Architektur-Agent (Analyst).** Pruft Konsistenz mit dem Standard hier. Memory-Modell-Tabelle korrekt? README-Sektionen alle da? Voice-Regeln eingehalten?
3. **Research-Agent.** Verifiziert Stand-der-Technik. Sind die Model-IDs aktuell? Gibt es n8n-Node-Versionen mit besseren Patterns? Sind die Provider-Pricing-Tabellen aktuell?

Mindestens 2 von 3 muessen GREEN sein. Bei einem AMBER muss das Finding entweder gefixt oder explizit dokumentiert sein (warum nicht gefixt).

Skill-Aufruf:

```
/agent-code-review /home/simple/n8n-templates
```

---

## Self-Check vor Commit

Wenn alle 10 Punkte ja sind, ist das Template / die Aenderung ready:

1. README enthaelt alle 12 Pflicht-Sektionen.
2. README enthaelt keine Em-Dashes (`grep ", " README.md` ergibt 0).
3. workflow.json valid (Validation-Script returns `Missing: NONE`).
4. workflow.json hat keine echten Credentials drin.
5. workflow.json hat alle Set-Me-Marker als sichtbare Sticky Notes.
6. cover.png existiert (oder cover.md hat einen verfizierten Flux-Prompt + Status).
7. Multi-Provider Pattern eingehalten (wenn LLM-Call drin).
8. Smoke-Test in echter n8n-Instanz gelaufen + Execution success.
9. Cross-Links zu anderen Templates im Footer korrekt + funktionieren.
10. Repo Top-Level-Files (CHANGELOG, ECOSYSTEM, etc.) auf Stand mit der Aenderung.

---

*Built by [StudioMeyer](https://studiomeyer.io) in Mallorca. Diese Brand-Bibel ist das interne Werkstatt-Manual. Aenderungen daran nur durch Matthias persoenlich nach Diskussion.*
