---
description: Run the full SEO refresh pipeline for a blog article
---

# /blog-refresh-run [taskId] [slug] [track]

Run the full SEO refresh pipeline for a blog article natively in Claude Code.
Phases: Research → Brief → Draft → QA → Google Doc → Task advance to editing/pending.

Arguments from the dispatch queue item: `taskId`, `slug`, `track` (A or B).

---

## Setup

Read these files before starting:
- `seo/blog-registry.json` — find the article entry for `slug`
- `social/tasks.json` — find the task entry for `taskId`
- `_context/brand-voice.md`
- `_context/audience.md`
- `seo/keyword-map.md`
- `_context/seo-content-inventory.md`

Set today's date: run `date +%Y-%m-%d`.

---

## Phase 1 — Research

Update task: `pipelineStage: 'research'`

**1a. Backup live CMS content**

Call `mcp__mangabeira-content__get_page` with `{ slug }`. Save the HTML content to:
`seo/backups/cms-backup-[slug]-[today].html`

**1b. Fetch original Google Doc** (if article has `googleDocId`)

Read the original doc with the `google-workspace` skill:
```bash
bash .claude/skills/google-workspace/scripts/doc-read.sh [googleDocId] > seo/backups/gdoc-[slug].txt
```
This is the primary source of the original article content.

**1c. Collect sources**

Write a JSON summary to `seo/refresh-sources-[slug]-[today].json`:
```json
{
  "slug": "[slug]",
  "track": "[track]",
  "date": "[today]",
  "sources": ["CMS backup", "Original Google Doc", "seo/keyword-map.md", "seo-content-inventory"]
}
```

Update task: add artifact `refresh_sources: 'seo/refresh-sources-[slug]-[today].json'`

---

## Phase 2 — Brief

Update task: `pipelineStage: 'writing'`

Using the original content, keyword map, and SEO inventory as context, write a refresh brief.

**Track A (Light Refresh):** Focus on updating stale data, filling content gaps vs competitors, fixing meta. Keep the article structure. Target 1,500–2,500 words total.

**Track B (Cornerstone Overhaul):** Full rewrite from a new angle. New keyword strategy, new H1, new structure. Target 2,500–4,000 words.

Brief must include:
- Strategic rationale (why this article needs this track)
- Primary keyword + 3–5 variants + GSC quick wins (position 5–20)
- Competitor gap analysis (what they cover that we don't)
- Article pivot: old framing → new framing, what to keep / drop / add
- Full H1 + H2 structure with word count targets per section
- AEO requirements: which H2s need Q&A block format (see Phase 3)
- Internal link targets (2–3 existing articles from seo-content-inventory)
- Hook differentiation note: scan `_context/seo-content-inventory.md` for existing article openers. The new article's meta_description and intro paragraph must not reuse the same sentence structure or "I analyzed X" pattern as any existing article.
- Image callout plan: for each H2, note what supporting image or data visualization would help (chart type, data to show, style)
- FAQ target list: 6–8 questions readers are likely searching related to this topic (these become the FAQ section)

Save brief to: `seo/briefs/refresh-[slug]-[today].md`

Update task artifact: `refresh_brief: 'seo/briefs/refresh-[slug]-[today].md'`

---

## Phase 3 — Draft

Write the full refreshed article following the brief exactly.

### Non-negotiable rules
- No em dashes — use periods or commas
- No banned phrases: leverage, game-changing, dive deep, unlock, seamlessly, revolutionary, groundbreaking, "in today's"
- First-person singular throughout: "I tracked", "I found", never "we"
- Hemingway Grade 9 or below
- Every factual claim tied to a real source (on-chain data, named protocols, published figures)
- 2–3 internal links to existing articles

### AEO: H2 title = statement + embedded question (required at every H2)

The H2 itself IS the question. Merge the topic statement and the search question into one heading:

```
## [Topic statement] and [Why/How does it cause X] / [What It Costs You]
```

Examples:
- "The Ethena Emission Curve Problem and Why It Guarantees Selling Pressure"
- "Farm-and-Dump Emission Schedules and How They Collapse Token Price"
- "Smart Contract Risk as a GTM Problem: Why Launching Unaudited Is a Disqualifier"

No separate Q: / A: labels. The first paragraph of each section IS the direct answer (40–60 words). No label needed. Just write it as a tight, direct-answer opening paragraph that works as a standalone snippet.

### Image callouts (required after each H2 body section)

After the body content of each H2 (before the next H2), insert an image recommendation callout. Use `|` as separator — never em dashes:

```
> 📊 **IMAGE:** [Type: chart/infographic/screenshot/diagram] | [Subject: what data or concept it shows] | [Data: specific numbers or axes if a chart] | [Style: dark background line chart / Dune Analytics aesthetic / branded color scheme] | [Aspect ratio: 16:9 for inline / 1:1 for social]
```

### Required sections at the end of the article

**FAQ section** (after the last content H2, before the author bio):

```markdown
## Frequently Asked Questions

**Q: [question 1 — high-search-volume phrasing]**

A: [Answer, 40–80 words. Conversational. First-person where natural.]

**Q: [question 2]**

A: [Answer]

... (6–8 total Q&A pairs)
```

**References section** (after FAQ, before author bio):

```markdown
## References

- [Source name] ([year]). [Finding cited in article]. [URL or publication name if available.]
- ...
```

Include every named source cited in the article body.

### Frontmatter

```yaml
---
title: "[SEO-optimised title]"
meta_description: "[150–160 chars — must NOT reuse the same opener pattern as any existing article. Check seo-content-inventory.md.]"
slug: [slug]
date_published: [today]
---
```

### Hook differentiation check

Before finalising the intro paragraph and meta_description, check `_context/seo-content-inventory.md` for the openers and meta descriptions of existing articles. The new article's first sentence and meta must be structurally distinct — different sentence type (statement vs. question vs. data point vs. scene-setting), different subject position. Never reuse "I analyzed X [articles/launches/protocols]" if another published article already opens that way.

Save draft to: `seo/[slug]-refresh-[today].md`

Update task artifact: `draft_blog: 'seo/[slug]-refresh-[today].md'`

---

## Phase 4 — QA

Run every check with an actual command. Do not eyeball. If a check fails, fix it before proceeding to Phase 5.

```bash
DRAFT="seo/[slug]-refresh-[today].md"

# 1. Word count
wc -w "$DRAFT"
# PASS: Track A ≥ 1,500 | Track B ≥ 2,500

# 2. Em dashes — MUST be zero
grep -c "—" "$DRAFT" && echo "HOLD: em dashes found" || echo "PASS: 0 em dashes"

# 3. Banned phrases
for p in "leverage" "game-changing" "dive deep" "seamlessly" "revolutionary" "groundbreaking" "in today's"; do
  grep -in "$p" "$DRAFT" && echo "HOLD: banned phrase '$p' found"
done

# 4. First-person check
grep -in "\bwe\b\|our team\|our clients" "$DRAFT" && echo "HOLD: first-person plural found" || echo "PASS"

# 5. H2 statement+question format
grep "^## " "$DRAFT"
# Manually verify: each H2 reads as "Statement and/Why/How..." — not a bare topic label

# 6. AEO direct answer openings
# Verify the paragraph immediately after each H2 is a direct 40-60 word answer

# 7. Image callouts present (one per H2 section)
grep -c "📊 \*\*IMAGE" "$DRAFT"
# PASS: count ≥ number of H2 sections (should be 8+)

# 8. Internal links — count + verify each one returns 200
grep -o '/publications/[^)]*' "$DRAFT" | sort -u | while read path; do
  status=$(curl -s -o /dev/null -w "%{http_code}" "https://mangabeira.net$path")
  echo "$path → $status"
  [ "$status" != "200" ] && echo "HOLD: broken internal link $path"
done

# 8b. External links — spot-check every reference URL returns 200 or 301
grep -oP 'https?://[^\s\)\"]+' "$DRAFT" | grep -v 'mangabeira.net' | sort -u | while read url; do
  status=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 "$url")
  echo "$url → $status"
  [ "$status" = "404" ] && echo "HOLD: broken external link $url"
done

# 9. FAQ section present with Q&A pairs
grep -c "^## Frequently Asked Questions" "$DRAFT"
grep -c "^\*\*Q:" "$DRAFT"
# PASS: FAQ section exists + ≥ 6 Q: lines

# 10. References section present
grep -c "^## References" "$DRAFT"

# 11. Hook differentiation — check meta_description is unique
head -5 "$DRAFT"
# Compare against existing articles in _context/seo-content-inventory.md

# 12. Frontmatter complete
grep "^title:" "$DRAFT"
grep "^meta_description:" "$DRAFT"
grep "^slug:" "$DRAFT"
```

Overall `qaStatus`: `passed` only when ALL checks return PASS. Any HOLD = fix first.

Write QA report to: `seo/qa-report-[slug]-[today].md`

Update task: `qaStatus: 'passed'|'hold'`

---

## Phase 5 — Editing Review Doc

Create a properly formatted Google Doc for Gabriel to read, annotate, and approve. **This is the editing gate only.** Design assets (images, protocol screenshots) are NOT added here.

Use the `google-doc-formatter` skill. Follow the skill template exactly — never substitute a manual batchUpdate or `google-docs_append_text`.

**Doc title:** `REFRESH REVIEW — [article title] — [today]`

**Document structure — content first, notes last (non-negotiable):**
```
[Article title — HEADING_1]
[Meta description — NORMAL_TEXT]
Slug: [slug]
Published: [date]
──────────────────────────────────────────────────────
[Article body — HEADING_2 for each H2, NORMAL_TEXT for paragraphs]
[Image callouts — NORMAL_TEXT bold after each section (placeholders only)]

## Frequently Asked Questions   ← HEADING_2
Q: [question]                   ← HEADING_3  (strip ** markers)
A: [answer]                     ← NORMAL_TEXT

## References                   ← HEADING_2
[Each reference — NORMAL_TEXT with working hyperlinks]

[Author bio — NORMAL_TEXT]
──────────────────────────────────────────────────────
── NOTES ──   ← HEADING_2
── BRIEF ──   ← HEADING_2  (## sub-sections → HEADING_3)
```

Key rules the script must follow (see full template in the skill):
1. **Strip all markdown syntax** — never let `**`, `[text](url)`, or `*` reach the doc
2. **Inline bold**: detect `**text**`, apply `updateTextStyle bold:true` to that range only
3. **Hyperlinks**: detect `[anchor](url)`, apply `updateTextStyle link:{url}` to that range
4. **Bold reset**: after every non-heading insert, send `updateTextStyle bold:false`
5. **FAQ questions** (`^**Q:`): strip `**`, apply `HEADING_3`
6. **FAQ answers** (`^A:`): `add_body_line()` — normal text with bold reset
7. **Relative URLs** (`/path`): resolve to `https://mangabeira.net/path`

Confirm the batchUpdate response contains `"replies"`. Print the doc URL.

---

## Phase 6 — Advance task to editing/pending

Update `social/tasks.json` for `taskId`:
```json
{
  "pipelineStage": "editing",
  "approvalStatus": "pending",
  "refreshPhase": "content",
  "qaStatus": "[passed|hold]",
  "artifacts": {
    "refresh_sources": "seo/refresh-sources-[slug]-[today].json",
    "refresh_brief": "seo/briefs/refresh-[slug]-[today].md",
    "draft_blog": "seo/[slug]-refresh-[today].md",
    "qa_report": "seo/qa-report-[slug]-[today].md"
  },
  "links": [
    { "type": "gdoc-refresh", "url": "[doc URL]", "label": "Editing Review Doc" }
  ]
}
```

Update `seo/blog-registry.json` for `slug`:
```json
{
  "status": "in-refresh",
  "lastRefreshTaskId": "[taskId]",
  "lastRefreshDate": "[today]"
}
```

Print: `[blog-refresh] Done. Task [taskId] → editing/pending | QA: [qaStatus] | Doc: [url]`

---

## Design Stage (triggered after editing approval)

This stage runs when Gabriel approves the editing review doc. It does not execute automatically — it begins when `pipelineStage` advances to `"design"`.

Update task: `pipelineStage: "design"`

---

## Phase 7 — Data Visualizations

Generate an image for every `> 📊 **IMAGE:**` callout in the draft.

Use nanobanana (`mcp__nanobanana__generate_image`): `model_tier: "pro"`, `resolution: "4k"`, `aspect_ratio: "16:9"`. Read each callout for chart type, axes, data, and aesthetic notes.

Save to: `seo/images/[slug]/img-[n]-[descriptor].png`

Also generate the hero image following the `og-taste` skill direction. Save to: `seo/images/[slug]-hero.png`

Update `seo/blog-registry.json` with `hero_image` and `hero_style`.

---

## Phase 8 — Protocol Screenshots

Identify the **3 protocols or tools most prominently referenced** in the article — those with their own H2 sections or cited with specific data points. Maximum 3. More creates visual clutter in the design doc.

For each:

**1. Take screenshot** (Tier 1 by default via `screenshot-taker` skill, key rotation on rate limit)
- Target: protocol homepage or most contextually relevant public page
- Viewport: 1440×900, above-fold only
- Params: `block_ads=true`, `block_cookie_banners=true`, `delay=2`

**2. QA the screenshot** — read the file visually before using it:
- No blocking popups, modals, or cookie banners covering content
- No 404, login wall, or error state
- Page content is clearly identifiable at ~480×270pt thumbnail size
- If homepage fails QA: try the protocol's docs, about page, or token analytics page instead

**3. Resize:** `sips -s format jpeg -s formatOptions 88 [input.png] --out /tmp/protocol-[name].jpg`

**4. Upload** to Supabase via mangabeira-content MCP `upload_image`. Filename: `protocol-[name].jpg`

**5. Insert into design review doc** after the HEADING_2 of the relevant section:
- GET document via gateway to find each heading's `endIndex`
- For each: `insertText "\n"` at `endIndex`, then `insertInlineImage` at `endIndex`
- Single batchUpdate, all insertions in **highest-index-first** order

---

## Phase 9 — Design Review Doc

Run `google-doc-formatter` skill. The script must follow the skill template exactly.

**Doc title:** `DESIGN REVIEW — [article title] — [today]`

**Review header block** (before article title):
```
DESIGN REVIEW
Status: APPROVED | QA: PASSED | Stage: Design Review | Date: [today]
──────────────────────────────────────────────────────
```

**Two-pass batchUpdate for images:**
- Pass 1: Text + formatting. Replace each image callout with `[IMAGE_N]` marker. Track each marker's `startIndex` and `len(marker)` at insert time.
- Pass 2 (reverse index order): For each marker — `insertInlineImage` at `startIndex`, then `deleteContentRange` from `startIndex+1` to `startIndex+1+len(marker)`.

Image dimensions: `480×270pt` (16:9) for all images.

---

## Phase 10 — Advance task to publish/pending

Update `social/tasks.json` for `taskId`:
```json
{
  "pipelineStage": "publish",
  "approvalStatus": "pending",
  "links": [
    { "type": "gdoc-refresh", "url": "[editing doc URL]", "label": "Editing Review Doc (Approved)" },
    { "type": "gdoc-design-review", "url": "[design doc URL]", "label": "Design Review Doc (Pending Approval)" }
  ],
  "artifacts": {
    "design": ["seo/images/[slug]-hero.png", "seo/images/[slug]/img-1-*.png", "..."]
  }
}
```

Print: `[blog-refresh-design] Done. Task [taskId] → publish/pending | Doc: [url]`