---
description: Final publish step for a blog article after Gate 2 approval
---

# /blog-publish-run [taskId] [slug]

Final publish step for a blog refresh. Runs after Gate 2 (final) approval.
Steps: Propagate design assets to Supabase + embed in EN draft → generate BR + ES translations → build schema for all 3 languages → publish all 3 in one CMS call → ping IndexNow → mark task done.

---

## Setup

Read:
- `social/tasks.json` — find task by `taskId`, get `artifacts.draft_blog`
- `seo/blog-registry.json` — find article by `slug`
- `_context/brand-voice.md`

Set today's date: `date +%Y-%m-%d`

Read the EN draft from `artifacts.draft_blog`. Parse the YAML frontmatter to extract `title` and `meta_description`. The body is everything after the closing `---`.

---

## Step 0 — Propagate design assets to Supabase + embed in EN draft

**Mandatory.** Every file in `task.artifacts.design[]` must be on Supabase Storage and embedded in the EN draft before translation runs. Translation reads from the embedded EN draft, so any asset missing here will be missing across all 3 languages.

For each path in `task.artifacts.design[]`:

1. Skip the hero (the file referenced by the page record's top-level `featured_image` is handled separately).
2. Derive a slug-prefixed object key for Supabase. Pattern: drop the article slug down to a short prefix (`defi-tl-` for `defi-token-launch-mistakes`, `solana-mev-` for `solana-mev-explained`, etc. — match whatever convention the article's existing assets use). Append the local file's distinguishing suffix:
   - Charts: `<prefix>img-<N>.jpg` (matches generated diagnostic chart IDs).
   - Protocol screenshots: `<prefix>proto-<name>.jpg`.
   - Other supporting graphics: `<prefix><descriptor>.jpg`.
3. Check whether the object already exists at `https://hetemmltaoirimmoxzku.supabase.co/storage/v1/object/public/blog-images/<key>` via a HEAD request. If 200, reuse. If 404, upload via `mcp__mangabeira-content__upload_image` (base64-encoded file contents). Record the returned `public_url`.
4. In the EN draft markdown, locate the placeholder for this asset. Placeholders follow the convention `> 📊 **IMAGE:** <description>` or `[IMAGE_<N>]`. Replace each placeholder with the live `<figure>` block matching the existing chart pattern in published articles:

   ```html
   <figure>
   <img src="<public_url>" alt="<descriptive alt>" width="<W>" height="<H>" style="width:100%;height:auto;border-radius:8px;">
   <figcaption><concise editorial caption></figcaption>
   </figure>
   ```

   Use the file's actual pixel dimensions for `width`/`height`. Localized `alt` and `figcaption` will be filled by the translator in Step 1.

   **NEVER add `loading="lazy"` to in-article images.** The mangabeira.net React SPA does not reliably trigger native lazy-loading inside its article scroll context, so any below-the-fold lazy image never loads (the reader sees blank gaps). Known-good published articles use no `loading` attribute. Confirmed + fixed 2026-06-02.
5. Append `public_url` to `task.artifacts.published[]`.

**Exit criterion:** every entry in `artifacts.design[]` (excluding the hero) has a corresponding entry in `artifacts.published[]` AND a `<figure>` block in the EN draft. If either side is short, do not proceed to Step 1.

Save the updated EN draft back to `artifacts.draft_blog`.

---

## Step 1 — Generate BR + ES translations

For each language (`br` = Brazilian Portuguese, `es` = Spanish Latin America):

Translate the full article. Rules:
- Natural translation, not literal. Preserve tone.
- Keep all technical terms unchanged: TVL, DeFi, NFT, Dune, DefiLlama, protocol names, ticker symbols.
- Keep all URLs unchanged.
- Translate the title and meta description.
- Translate the URL slug: e.g. "defi-token-launch-mistakes" → "erros-lancamento-token-defi" (BR) or "errores-lanzamiento-token-defi" (ES).
- Keep all markdown formatting unchanged.

Save translations:
- `seo/[slug]-refresh-[today]-br.md`
- `seo/[slug]-refresh-[today]-es.md`

---

## Step 2 — Build schema for all 3 languages

For each language, build a JSON-LD schema object with these nodes:

**Article node:**
- `@type: Article`
- `@id: [url]#article`
- `headline`: translated title
- `description`: translated meta_description
- `dateModified`: today's date + T12:00:00+00:00
- `datePublished`: preserve from existing CMS page if available
- `author`: `{ "@id": "https://mangabeira.net/#person" }`
- `publisher`: `{ "@id": "https://mangabeira.net/#organization" }`
- `inLanguage`: `en-US` / `pt-BR` / `es-LA`
- `wordCount`: count words in the EN draft

**BreadcrumbList node** with localized labels:
- EN: Home / Publications / [title]
- BR: Início / Artigos / [title]
- ES: Inicio / Artículos / [title]

**Person node** (`@id: https://mangabeira.net/#person`): Gabriel Mangabeira, Web3 Growth Strategist

**Organization node** (`@id: https://mangabeira.net/#organization`): Mangabeira / Analyst in the Arena

**WebPage node** with the page URL and dates.

URL structure per language:
- EN: `https://mangabeira.net/publications/[en-slug]`
- BR: `https://mangabeira.net/br/artigos/[br-slug]`
- ES: `https://mangabeira.net/es/articulos/[es-slug]`

If `seo/schema/[slug]-schema.json` exists, preserve any `FAQPage` or `HowTo` nodes from it and merge them into the EN schema.

Save updated EN schema to: `seo/schema/[slug]-schema.json`

---

## Step 3 — Fetch live page for preserved fields

Call `mcp__mangabeira-content__get_page` with `{ slug }`.
Preserve: `category`, `featured_image`, original `date_published` from the EN translation.

---

## Step 4 — Publish all 3 translations in ONE call

Call `mcp__mangabeira-content__upsert_page` with:

```json
{
  "slug": "[en-slug]",
  "category": "[from live page, default: blog]",
  "status": "published",
  "author_name": "Gabriel Mangabeira",
  "featured_image": "[from live page if set]",
  "translations": [
    {
      "language": "en",
      "title": "[EN title]",
      "meta_description": "[EN meta]",
      "content": "[EN body]",
      "localized_slug": "[en-slug]",
      "schema": { "[EN schema object]" }
    },
    {
      "language": "br",
      "title": "[BR title]",
      "meta_description": "[BR meta]",
      "content": "[BR body]",
      "localized_slug": "[br-slug]",
      "schema": { "[BR schema object]" }
    },
    {
      "language": "es",
      "title": "[ES title]",
      "meta_description": "[ES meta]",
      "content": "[ES body]",
      "localized_slug": "[es-slug]",
      "schema": { "[ES schema object]" }
    }
  ]
}
```

CRITICAL: all 3 translations in one call. Separate calls wipe previous translations.

---

## Step 5 — Ping IndexNow

Ping these 3 URLs via `mcp__bing-webmaster__*` or direct fetch to `https://api.indexnow.org/IndexNow` (POST) and `https://www.bing.com/indexnow` (POST):

- `https://mangabeira.net/publications/[en-slug]`
- `https://mangabeira.net/br/artigos/[br-slug]`
- `https://mangabeira.net/es/articulos/[es-slug]`

Use `INDEXNOW_KEY` from `.env` if set. If not set, skip (CMS triggers IndexNow on publish anyway).

---

## Step 6 — Mark done

Update `social/tasks.json` for `taskId`:
```json
{
  "pipelineStage": "done",
  "approvalStatus": "not-needed",
  "refreshPhase": "done",
  "completed": "[today]",
  "artifacts": {
    "translation_br": "seo/[slug]-refresh-[today]-br.md",
    "translation_es": "seo/[slug]-refresh-[today]-es.md",
    "published": [
      "https://mangabeira.net/publications/[en-slug]",
      "https://mangabeira.net/br/artigos/[br-slug]",
      "https://mangabeira.net/es/articulos/[es-slug]"
    ]
  }
}
```

Update `seo/blog-registry.json` for `slug`:
```json
{
  "status": "published",
  "lastUpdated": "[today]",
  "lastRefreshDate": "[today]",
  "lastUpdatedSource": "refresh"
}
```

Append to `reports/publication-log.md`:
```
| [today] | Blog refresh | [article title] | https://mangabeira.net/publications/[slug] | EN + BR + ES |
```

Print: `[blog-publish] Done. Published EN + BR + ES | [en-url]`