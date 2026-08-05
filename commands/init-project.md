---
description: Initialize a new MangaOS project workspace from an onboarding questionnaire
---

# /init-project

Initialize a new MangaOS project workspace from a filled `onboarding.md` questionnaire. Run from the project root directory.

> **Recommended first step:** Before running this command, run `TASK_SERVER_PORT=[project port] npm run onboarding` and open `http://localhost:[project port]/onboarding` for a 5-minute voice interview that auto-fills most sections of `onboarding.md`. After the interview, complete the Visual Identity and Technical Setup sections manually, then run `/init-project`.

---

## WHAT THIS COMMAND DOES

Reads `onboarding.md`, validates required fields, generates all `_context/` files, initializes `campaigns/campaigns.json`, creates the full folder structure, seeds `research/cycle-brief-current.md`, updates `CLAUDE.md`, and appends to `docs/decision-log.md`. Outputs a post-init checklist for manual steps.

---

## EXECUTION STEPS

Follow these steps in sequence. Do not skip any step. Do not ask for permission between steps — execute all steps autonomously and report what was done at the end.

---

### STEP 1 — Read onboarding.md

Read the file `onboarding.md` from the current working directory (project root). If the file does not exist, stop and output:

```
ERROR: onboarding.md not found in the current directory.
Run this command from the project root, and make sure onboarding.md exists.
```

---

### STEP 2 — Validate required fields

Check that the following critical fields are filled (not empty, not "TBD"):

**Critical fields (hard stop if missing):**
- Project name
- Project slug
- Type (must be one of: `personal-brand`, `saas-app`, `publisher`, `ai-agent-persona`, `media-brand`)
- Primary language
- One-line positioning
- Tone (3 adjectives)
- Primary color (hex)
- Headline font
- ICP 1 name
- ICP 1 pain point
- ICP 1 trigger
- Campaign name
- Campaign goal
- Primary KPI
- Campaign timeline

**Soft warnings (proceed but flag):**
- Primary URL (if missing, warn "Primary URL not set — update CLAUDE.md when known")
- Analytics property ID
- Search Console property URL
- Task server port

If any **critical fields** are missing or still say "TBD", list them and output:

```
VALIDATION FAILED — missing critical fields:
- [field name]
- [field name]

Fill these fields in onboarding.md before re-running /init-project.
```

Then stop. Do not proceed to Step 3.

If all critical fields are present, proceed. Print any soft warnings above the output, then continue.

---

### STEP 3 — Generate `_context/brand-voice.md`

Create or overwrite `_context/brand-voice.md`. Use this exact structure, filling each section from the corresponding onboarding.md fields:

```markdown
# Brand Voice
# [Project name]

## Positioning
[One-line positioning]

## Tone
[Three adjectives from onboarding, written as: "Adjective, adjective, adjective."]

## Voice Principles
- [Derive 3-5 principles from the tone adjectives and writing rules. Each is one sentence.]

## Vocabulary

### Always use
- [Each term from "Vocabulary to always use", one per bullet]

### Never use
- [Each term from "Vocabulary to NEVER use", one per bullet]

## Writing Rules
[Writing rules from onboarding, formatted as a bulleted list]

## Sentence Length
[Sentence length target from onboarding]

## Banned Phrases (AI clichés)
- "revolutionary"
- "game-changing"
- "seamless"
- "leverage" (as a verb)
- "unlock"
- "delve"
- "it's worth noting"
- "in conclusion"
- [Add any project-specific banned phrases from the Never Use list that are full phrases]

## Content Examples
[Leave blank — add sample posts/articles after first content is produced]
```

---

### STEP 4 — Generate `_context/audience.md`

Create or overwrite `_context/audience.md`. Use this structure:

```markdown
# Audience
# [Project name]

## Primary ICP: [ICP 1 name]

**Who they are:** [ICP 1 description]
**Pain point:** [ICP 1 pain]
**Trigger event:** [ICP 1 trigger]
**Where they spend time:** [ICP 1 channels, if filled; otherwise omit]
**Content they trust:** [ICP 1 content types, if filled; otherwise omit]

---

## Secondary ICP: [ICP 2 name, if provided]

**Who they are:** [ICP 2 description]
**Pain point:** [ICP 2 pain]
**Trigger event:** [ICP 2 trigger]
**Where they spend time:** [ICP 2 channels, if filled; otherwise omit]

---

[Repeat for ICP 3 if present]

## Excluded Audiences
[From onboarding if provided; otherwise write "Not defined — update as targeting is refined."]

## Channel Preferences by ICP
| ICP | Primary channel | Secondary channel |
|-----|----------------|-------------------|
| [ICP 1 name] | [from onboarding or infer from type] | [from onboarding or infer] |
| [ICP 2 name] | [from onboarding or infer] | [from onboarding or infer] |
```

---

### STEP 5 — Generate `_context/product-info.md`

Create or overwrite `_context/product-info.md`. Use this structure:

```markdown
# Product Info
# [Project name]

## What it is
[Short description from onboarding]

## Stack / Platform
[Stack/platform from onboarding]

## Primary URL
[Primary URL, or "TBD — update when live"]

## Core Offering
[Core offering from onboarding Product/Content Details section]

## Primary CTA
[Primary CTA from onboarding]

## Lead Magnet / Hook
[Lead magnet from onboarding]

## Key Differentiators
[Key differentiators from onboarding, as bullets]

## Pricing / Access Model
[From onboarding if provided; otherwise "Not yet defined"]

## Monetization
[From onboarding if provided; otherwise "Not yet defined"]

## Content Categories
[From onboarding if provided; otherwise omit section]

## Publishing Frequency
[From onboarding if provided; otherwise omit section]

## Proof Points / Social Proof
[Leave blank — add after first results are available]
```

---

### STEP 5b — Generate `_context/positioning.md`

Create or overwrite `_context/positioning.md`. Pull from the new Positioning section of onboarding.md. If the Positioning section is absent or blank, derive a draft from the one-line positioning, ICP 1, and key differentiators — and flag it as "DRAFT — fill Positioning section in onboarding.md to finalize."

```markdown
# Positioning
# [Project name]

## Full Positioning Statement
[Full positioning statement from onboarding, or derived draft]

## Primary Message
[Primary message — the one thing the brand must communicate above all else]

## Supporting Messages
- [Secondary message 1]
- [Secondary message 2]
- [Secondary message 3, if provided]

## Category
[Category name — how this brand wants to be categorized in the market]

## Primary Alternative
[What people use today instead of this]

## Positioning Guardrails — What We Never Claim
[What to never claim, as a bulleted list]

## Proof Points
[Leave blank — add after first results, case studies, or testimonials are available]
```

---

### STEP 6 — Generate `_context/style-guide.md`

Create or overwrite `_context/style-guide.md`. This is a seed file — it will be refined in the Design System phase of onboarding (after research phases) using the `visual-references` skill vision board. For now, fill from onboarding data.

Use this structure:

```markdown
# Style Guide
# [Project name]

## Color Palette

| Role | Hex | Usage |
|------|-----|-------|
| Primary | [primary color hex] | CTAs, headlines, key UI |
| Secondary | [secondary color hex] | Supporting elements |
| Accent | [accent color hex] | Highlights, badges |
| Background | [background color hex] | Page/card backgrounds |
| Text | [text color or infer] | Body text |

## Typography

| Role | Font | Weight |
|------|------|--------|
| Headline | [headline font] | Bold (700) |
| Subheadline | [headline font] | SemiBold (600) |
| Body | [body font] | Regular (400) |
| Caption | [body font] | Light (300) |

## Visual Style
[Visual style from onboarding]

## Logo
[Logo file path from onboarding, or "Not yet finalized — update when available"]

## Image Direction
[Infer 2-3 image direction rules from the visual style and tone. E.g., "No stock photo smiles. Data visualizations preferred. Dark backgrounds with neon accents for hero sections."]

## Social Graphic Specs

| Format | Dimensions | Notes |
|--------|-----------|-------|
| LinkedIn single image | 1200×627px | Primary color background, white text |
| LinkedIn carousel slide | 1080×1080px | Consistent header across slides |
| X/Twitter post image | 1600×900px | High contrast, readable at thumb size |
| Newsletter header | 600×200px | Clean, no clutter |

## Brand Don'ts
- No gradient abuse
- No clipart or generic stock photography
- No font mixing beyond the two-font system
- [Add any visual rules from the "Never use" or tone context]
```

---

### STEP 7 — Initialize `campaigns/campaigns.json`

Create the `campaigns/` directory if it doesn't exist. Create or overwrite `campaigns/campaigns.json` with one campaign object derived from the First Campaign section of onboarding.md:

```json
{
  "campaigns": [
    {
      "slug": "[campaign-name-slugified]",
      "name": "[Campaign name]",
      "status": "draft",
      "goal": "[Campaign goal]",
      "kpi": "[Primary KPI]",
      "channels": ["[channel1]", "[channel2]"],
      "cta": "[CTA for this campaign]",
      "leadMagnet": "[Lead magnet or hook]",
      "startDate": "[Timeline start date]",
      "endDate": "[Timeline end date]",
      "createdAt": "[today's date in YYYY-MM-DD]",
      "brief": "campaigns/[campaign-slug]/brief.md",
      "assets": "campaigns/[campaign-slug]/assets.md",
      "tags": ["[project type]", "[primary language]"]
    }
  ],
  "lastUpdated": "[today's date]"
}
```

Then create `campaigns/[campaign-slug]/` folder and write a seed `campaigns/[campaign-slug]/brief.md`:

```markdown
# Campaign Brief: [Campaign name]

**Status:** Draft
**Created:** [today's date]
**Goal:** [Campaign goal]
**KPI:** [Primary KPI]
**Timeline:** [start] to [end]
**Channels:** [channels list]
**CTA:** [CTA]
**Lead Magnet:** [Lead magnet]

## ICP Target
[ICP 1 name and pain point — the audience this campaign reaches]

## Core Message
[Derive from one-line positioning + campaign goal. One paragraph.]

## Content Plan
[Leave as TBD — fill after first cycle brief is approved]

## Approval Gate
- [ ] Brief reviewed by Gabriel
- [ ] Content plan approved
- [ ] Assets approved before publish
```

---

### STEP 8 — Seed `social/tasks.json`

Create `social/` directory if it doesn't exist.

If `social/tasks.json` already exists with tasks in it, read it and update only the `lastUpdated` field to today's date. Write it back. Do not overwrite existing tasks.

If `social/tasks.json` does not exist or has an empty `tasks` array, create it with 3 seed task objects derived from the first campaign in onboarding.md. Use this structure:

```json
{
  "lastUpdated": "[today's date]",
  "schema": "2.1",
  "tasks": [
    {
      "id": "[PROJECT-SLUG]-001",
      "title": "Approve campaign brief: [Campaign name]",
      "status": "todo",
      "priority": "high",
      "channel": "ops",
      "campaign": "[campaign-slug]",
      "dueDate": "[campaign start date]",
      "approvalStatus": "pending",
      "qaStatus": "not-run",
      "statusChangedDate": "[today's date]",
      "files": ["campaigns/[campaign-slug]/brief.md"],
      "notes": "Review and approve the campaign brief before content production begins. Set status to 'approved' when ready."
    },
    {
      "id": "[PROJECT-SLUG]-002",
      "title": "Produce first content asset for [Campaign name]",
      "status": "todo",
      "priority": "high",
      "channel": "[first channel from campaign channels list]",
      "campaign": "[campaign-slug]",
      "dueDate": "[campaign start date + 3 days]",
      "approvalStatus": "not-needed",
      "qaStatus": "not-run",
      "statusChangedDate": "[today's date]",
      "files": [],
      "notes": "First content piece for this campaign. Requires campaign brief approval (task [PROJECT-SLUG]-001) before starting."
    },
    {
      "id": "[PROJECT-SLUG]-003",
      "title": "Distribution + review: [Campaign name] launch week",
      "status": "todo",
      "priority": "medium",
      "channel": "ops",
      "campaign": "[campaign-slug]",
      "dueDate": "[campaign start date + 7 days]",
      "approvalStatus": "not-needed",
      "qaStatus": "not-run",
      "statusChangedDate": "[today's date]",
      "files": ["research/cycle-brief-current.md"],
      "notes": "End-of-week distribution review. Check engagement metrics, update cycle brief with what worked, plan next week."
    }
  ]
}
```

Substitute all `[PROJECT-SLUG]`, `[Campaign name]`, `[campaign-slug]`, and date fields with real values from onboarding.md.

---

### STEP 9 — Create folder structure

Create all of the following directories if they do not already exist. Use `mkdir -p` for each. Do not delete or overwrite any existing files.

```
social/linkedin/
social/x/
social/newsletter/
seo/
seo/briefs/
seo/schema/
seo/monitoring/
seo/research/
research/
reports/
ads/
pages/
videos/
docs/
_templates/
_templates/references/
_templates/references/screenshots/social-post/
_templates/references/screenshots/carousel/
_templates/references/screenshots/landing-page/
_templates/references/screenshots/dashboard/
_templates/references/screenshots/deck/
_templates/references/screenshots/video-thumbnail/
_templates/references/screenshots/ad-creative/
_templates/references/screenshots/email-header/
_templates/references/screenshots/og-image/
_templates/references/screenshots/profile-asset/
campaigns/
.claude/
.claude/commands/
.claude/agents/
.claude/rules/
.claude/skills/
.claude/agent-memory/
_ops/
_context/
```

---

After creating folders, copy the reference library stubs from the project template:
```
~/Documents/Gabriel\ Mangabeira/marketing/_templates/project-template/_templates/references/library.json → ./_templates/references/library.json
~/Documents/Gabriel\ Mangabeira/marketing/_templates/project-template/_templates/references/index.html → ./_templates/references/index.html
```

Then update `_templates/references/library.json` to set `"project"` to the project name from onboarding.md.

Log: "CREATED: _templates/references/ (library.json + index.html + 10 format folders)"

---

### STEP 9b — Copy type-mapped SOPs

Read the project type from onboarding.md. Copy the following SOP files from `~/Documents/Gabriel\ Mangabeira/marketing/_sop/` into `./_sop/`. If the source file does not exist, log a warning and continue.

| Project type | SOPs to copy |
|---|---|
| `saas-app` | sop-02, sop-03, sop-05, sop-07, sop-09, sop-13 |
| `personal-brand` | sop-01, sop-02, sop-03, sop-04, sop-05, sop-06, sop-07, sop-08, sop-09, sop-10, sop-11, sop-12, sop-13 |
| `publisher` | sop-02, sop-03, sop-05, sop-07, sop-12 |
| `ai-agent-persona` | sop-03, sop-05, sop-09, sop-17 |
| `media-brand` | sop-03, sop-05, sop-07, sop-11, sop-16 |

Each SOP filename follows the pattern `sop-NN-[name].md`. Copy by matching the `sop-NN` prefix — do not hardcode full filenames. For each SOP number in the list, find the file matching `sop-NN-*.md` in the source folder and copy it.

Log each copied file: "COPIED: _sop/[filename]". Log any misses: "WARN: sop-NN not found in reference workspace — add manually."

---

### STEP 9c — Copy standard agents and skills reference

Copy the 6 standard agent files from `~/Documents/Gabriel\ Mangabeira/marketing/.claude/agents/` into `./.claude/agents/`:
- `content-writer.md`
- `data-analyst.md`
- `designer.md`
- `marketing-systems-engineer.md`
- `research-agent.md`
- `seo-agent.md`

Do NOT copy `token-health-scan-am.md` or any other account manager files — these belong in the central marketing workspace only.

Copy the skills reference table:
```
~/Documents/Gabriel\ Mangabeira/marketing/.claude/rules/skills-reference.md
→ ./.claude/rules/skills-reference.md
```

Log each copied file: "COPIED: .claude/agents/[filename]" and "COPIED: .claude/rules/skills-reference.md".

---

### STEP 10 — Write `research/cycle-brief-current.md`

Create `research/cycle-brief-current.md` as a seed file:

```markdown
# Cycle Brief — [Campaign name]
**Status:** Seed (not yet approved)
**Created:** [today's date]
**Campaign:** [Campaign slug]

## This Cycle's Goal
[Campaign goal]

## Primary KPI
[Primary KPI]

## Target Audience This Cycle
[ICP 1 name] — [ICP 1 pain point]

## Content Themes
[Derive 3 content themes from the campaign goal, ICP pain, and one-line positioning. Each is one sentence.]

## Channels This Cycle
[Channels from campaign]

## CTA
[Campaign CTA]

---
*This is a seed brief. Replace with the approved cycle brief before content production begins.*
```

---

### STEP 10b — Configure `dashboard/dashboard.config.json`

Read `onboarding.md` to extract the project name, slug, primary color, and task server port. Then write `dashboard/dashboard.config.json`:

```json
{
  "projectName": "[Project name from onboarding]",
  "projectSlug": "[Project slug from onboarding]",
  "port": [dashboard port — use task server port + 1 if no dedicated dashboard port is set, or use 3006+ for new projects],
  "primaryColor": "[Primary color hex from onboarding]",
  "logoEmoji": "[Pick a single emoji that fits the project type: 💎 for SaaS, 🤖 for agent-persona, 📡 for media-brand, 🔵 for personal-brand, 📰 for publisher]",
  "description": "[Brief description — derive from project type: e.g., 'SaaS Marketing Ops', 'Agent Persona Ops', 'Media Brand Ops', 'Growth Ops', 'Publisher Ops']"
}
```

After writing the file, verify it is valid JSON by attempting to parse it. If parsing fails, fix the file before continuing.

---

### STEP 11 — Update `CLAUDE.md`

Read `CLAUDE.md` from the project root. Find the WHO YOU ARE section. Replace the placeholder text (if any) with:

```
You are the AI marketing system for [Project name].

[One-line positioning from onboarding.md]

Your role is to execute marketing tasks with minimal friction, produce finished deliverables (not drafts), and route outputs to the correct working folder. You do not ask clarifying questions if the answer is in the context files. You do not produce generic content. Everything you output must be calibrated to this brand.
```

If `CLAUDE.md` does not exist, create it from the template at `~/Documents/Gabriel\ Mangabeira/marketing/_templates/project-template/CLAUDE.md` if that file exists. Otherwise create a minimal `CLAUDE.md` with:

```markdown
# CLAUDE.md
# [Project name] Marketing Workspace

## WHO YOU ARE

You are the AI marketing system for [Project name].

[One-line positioning]

Your role is to execute marketing tasks with minimal friction, produce finished deliverables (not drafts), and route outputs to the correct working folder. You do not ask clarifying questions if the answer is in the context files. You do not produce generic content. Everything you output must be calibrated to this brand.

## CONTEXT FILES

Load these before any task:
- `_context/brand-voice.md` — before writing any content
- `_context/audience.md` — before targeting any audience
- `_context/product-info.md` — before referencing products, CTAs, or proof points
- `_context/style-guide.md` — before producing any visual or formatted output

## WORKSPACE STRUCTURE

See ~/Documents/Gabriel\ Mangabeira/marketing/CLAUDE.md for the full MangaOS architecture.
This workspace follows the same structure and SOPs as the reference workspace.
```

---

### STEP 12 — Append to `docs/decision-log.md`

Create `docs/` directory if it doesn't exist. Append the following entry to `docs/decision-log.md` (create the file if it doesn't exist):

```markdown
---

## DEC-001 — Project initialized via /init-project

**Date:** [today's date]
**Decision:** [Project name] marketing workspace initialized via the /init-project command from onboarding.md.
**Rationale:** Standardized initialization ensures all MangaOS context files, folder structure, and campaign objects are present and consistent from day one.
**Alternatives considered:** Manual setup (rejected — error-prone and slow).
**Impact:** All _context/ files generated, campaigns/campaigns.json seeded, folder structure created, cycle-brief-current.md seeded.
**Supersedes:** N/A
**Superseded by:** N/A
```

---

### STEP 13 — Log soft warnings + output the Post-Init Checklist

**First:** If any soft warning fields were flagged in Step 2 (Primary URL, GA4 ID, GSC property, Task server port), append a single `WARN-001` entry to `docs/decision-log.md`:

```markdown
---

## WARN-001 — Soft warnings at initialization

**Date:** [today's date]
**Fields not set at init time:** [comma-separated list of blank soft-warning fields]
**Action required:** Fill these fields in onboarding.md and re-run the relevant /init-project steps, or update the generated files directly when values are available.
```

If no soft warnings exist, skip this entry.

**Then:** Print the following checklist as the final output. Do not skip this step.

```
=====================================================
PROJECT INITIALIZED: [Project name]
Date: [today's date]
=====================================================

Context files generated:
  ✓ _context/brand-voice.md
  ✓ _context/audience.md
  ✓ _context/product-info.md
  ✓ _context/style-guide.md
  ✓ _context/positioning.md

Campaign initialized:
  ✓ campaigns/campaigns.json ([Campaign name] — draft)
  ✓ campaigns/[campaign-slug]/brief.md

Workspace seeded:
  ✓ social/tasks.json (3 starter tasks)
  ✓ _sop/ (type-mapped SOPs copied from reference workspace)
  ✓ .claude/agents/ (6 standard agent files copied)
  ✓ .claude/rules/skills-reference.md (copied)
  ✓ research/cycle-brief-current.md
  ✓ docs/decision-log.md (DEC-001)
  ✓ Folder structure created

[Any soft warnings from Step 2 listed here]

=====================================================
MANUAL STEPS REQUIRED (complete in order):
=====================================================

[ ] 1. Copy task server files:
        cp ~/Documents/Gabriel\ Mangabeira/marketing/task-server.js ./task-server.js
        cp ~/Documents/Gabriel\ Mangabeira/marketing/tasks.html ./tasks.html

[ ] 2. Copy environment variables from reference workspace:
        cp ~/Documents/Gabriel\ Mangabeira/marketing/.env ./.env
        (The shared API keys — Apify, OpenRouter, Google Workspace OAuth, etc. — are already included via the marketing/.env copy.)

[ ] 3. Wire internal data access — create + populate project-specific integrations in .env:
        - GA4: Create property at analytics.google.com → Admin → Create Property → add GA4_PROPERTY_ID=
        - GSC: Add property at search.google.com/search-console → verify ownership → add GSC_PROPERTY_URL=
        - CMS: If project has its own CMS (Supabase, custom) → CONTENT_CMS_URL= + CONTENT_MCP_SECRET=
        - Newsletter: If using Beehiiv → BEEHIIV_API_KEY= + BEEHIIV_PUBLICATION_ID=
        - Any other project-specific API (Stripe, custom data source) → add to .env
        Restart the Claude Code window after updating so all MCP tools (GA4, GSC) pick up the new keys.
        Note: GA4 + GSC properties can be created now even before the domain is live — no data yet, but connections are ready.

[ ] 4. Add project to multi-project dashboard:
        Edit ~/Documents/Gabriel\ Mangabeira/marketing/dashboard/projects.json
        Add: { "name": "[Project name]", "slug": "[slug]", "port": [task server port], "path": "[absolute path to project root]" }

[ ] 5. Initialize git:
        git init && git add . && git commit -m "init: [project name] MangaOS workspace"

[ ] 6. Set the project port in `.env`:
        TASK_SERVER_PORT=[task server port from onboarding]

[ ] 7. Update _context/product-info.md with live URLs once deployed

=====================================================
NEXT STEPS — ONBOARDING PHASES (type-specific, run in order):
=====================================================

These phases run after /init-project completes. Each has a gate — do not start the next phase
until the current phase output is approved by the project owner.

Read the project type from onboarding.md and follow the matching track below.
Project types: saas-app | personal-brand | ai-agent-persona | media-brand | publisher

-----------------------------------------------------
TYPE: saas-app
-----------------------------------------------------

PHASE 1 — Strategic Foundation
  Run: /gtm-foundations
  Focus: OPE Canvas (is this a SaaS to scale or a lead magnet?), value prop canvas,
         90-day GTM plan, 5 custom analytics events to wire before launch,
         pricing model decision, LinkedIn + email strategy
  Output: outputs/01-phase-1-summary.md
  Gate: Project owner approves 90-day GTM plan and pricing model
  Effort: 4-6 hrs

PHASE 2 — Competitor + Audience Intelligence (deep research, run in parallel)
  A. ICP research: research-agent → "run audience-intelligence-brief for [project name]"
     Uses: openrouter-research (Grok + Perplexity + Gemini)
     Output: research/audience-insights.md + HTML presentation
  B. Competitor tool landscape: research-agent → "competitive tool scan for [project name] — what
     tools exist in this space, what signals/features do they expose, what is missing?"
     Uses: firecrawl, openrouter-research
     Output: research/competitive-landscape-tools-YYYY-MM-DD.md
  C. LinkedIn content competitors: linkedin-competitor-analysis skill
     Scope: who posts content in this product's vertical, not Gabriel's personal brand competitors
     Output: research/competitor-analysis-linkedin-YYYY-MM-DD.html
  Gate: Competitive gap matrix approved + audience brief exec summary approved
  Effort: 3-5 hrs

PHASE 3 — Internal Product Audit (requires live URL + Phase 2 benchmarks)
  A. Technical + SEO audit: seo-agent → "SEO onboarding" (seo-aeo-onboarding skill)
     Critical check: are dynamic pages (e.g. /scan/[token]) SSR or client-side only?
     Output: seo/onboarding/YYYY-MM-DD-technical-audit.md
  B. Signal/feature audit: validate product's core signals against competitor gap matrix from Phase 2
     Output: research/signal-audit-YYYY-MM-DD.md
  C. UX flow audit: test as each ICP using screenshot-taker skill
     Output: research/ux-flow-audit-YYYY-MM-DD.md
  Gate: No launch-blocking issues in any audit output
  Effort: 3-4 hrs

PHASE 4 — Design System + Template Library
  A. Visual influences research: designer agent → visual-references skill
     Input: _context/style-guide.md (colors, tone adjectives, visual style)
     Goal: vision board of references matching the product's aesthetic (app UI, data dashboards,
           fintech/crypto tools as appropriate)
     Output: _templates/vision-board-[slug].html
  B. Design system lock-in
     Refine _context/style-guide.md with concrete direction from the vision board:
     image direction rules, component patterns, brand don'ts
  C. Social media templates: designer agent → social-graphics + canvas-design
     Generate base templates for all active channels:
     - LinkedIn single image (1200×627px): text-card + data-card variants
     - LinkedIn carousel (1080×1080px): cover slide + body slide template
     - X/Twitter post image (1600×900px): text-card variant
     - Newsletter header (600×200px) — only if newsletter is a channel
     Save to: _templates/social-creatives/
  D. Deck/presentation template: branded-deck skill (if decks are a deliverable channel)
     Output: _templates/decks/[slug]-deck-template.pptx
  Gate: Gabriel approves vision board + full template set before content production starts
  Effort: 2-4 hrs

PHASE 5 — GTM Plan + Content Engine Activation
  A. Keyword calendar: seo-agent → "keyword-calendar for [project name]"
  B. Approve campaign brief → set status "draft" → "active" in campaigns.json
  C. Approve cycle brief in research/cycle-brief-current.md
  D. First content pieces: launch data post (LinkedIn + X) — use real product data
  E. Wire analytics: GA4_PROPERTY_ID + GSC_PROPERTY_URL in .env once live
  Gate: Campaign active + cycle brief approved + first content piece drafted
  Effort: 4-6 hrs

Total effort: 19-30 hrs

-----------------------------------------------------
TYPE: personal-brand
-----------------------------------------------------

PHASE 1 — Strategic Foundation
  Run: /gtm-personal-brand
  Focus: four-phase sprint roadmap (Foundation → Activation → Outreach → Paid/Events),
         exchange-rate pricing if LATAM-based, lead magnet sequencing, discovery call funnel
  Output: outputs/01-phase-1-summary.md
  Gate: Sprint roadmap and pricing approved
  Effort: 3-5 hrs

PHASE 2 — Audience + Competitor Intelligence (deep research, run in parallel)
  A. ICP research: research-agent → "run audience-intelligence-brief for [project name]"
     Focus: consulting/fractional buyer personas, what triggers hiring a fractional exec
     Output: research/audience-insights.md
  B. LinkedIn competitor scan: linkedin-competitor-analysis skill
     Focus: direct personal brand competitors in the same vertical
     Output: research/competitor-analysis-linkedin-YYYY-MM-DD.html
  C. Content audit: what content formats and topics drive inbound for competitors?
     Uses: apify-mcp (LinkedIn scrape), openrouter-research
     Output: research/content-audit-competitors-YYYY-MM-DD.md
  Gate: Competitive gap and content angle gaps identified and approved
  Effort: 2-4 hrs

PHASE 3 — SEO + Content Baseline
  A. SEO onboarding: seo-agent → "SEO onboarding" (seo-aeo-onboarding skill)
  B. Keyword calendar: seo-agent → "keyword-calendar"
  Gate: Topical authority map approved + pillar structure defined
  Effort: 2-3 hrs

PHASE 4 — Design System + Template Library
  A. Visual influences research: designer agent → visual-references skill
     Input: _context/style-guide.md (colors, tone adjectives, visual style)
     Goal: vision board of references matching the personal brand aesthetic (consulting,
           thought leadership, professional services in the relevant vertical)
     Output: _templates/vision-board-[slug].html
  B. Design system lock-in
     Refine _context/style-guide.md with concrete direction from the vision board:
     image direction rules, headline treatment, brand don'ts
  C. Social media templates: designer agent → social-graphics + canvas-design
     Generate base templates for all active channels:
     - LinkedIn single image (1200×627px): text-card + quote-card variants
     - LinkedIn carousel (1080×1080px): cover slide + body slide template
     - X/Twitter post image (1600×900px): text-card variant
     - Newsletter header (600×200px) — only if newsletter is a channel
     Save to: _templates/social-creatives/
  D. Consulting deck template: branded-deck skill
     Output: _templates/decks/[slug]-consulting-deck-template.pptx
  Gate: Gabriel approves vision board + full template set before content production starts
  Effort: 2-4 hrs

PHASE 5 — Content Engine Activation
  A. Approve cycle brief
  B. First LinkedIn post + blog article per cycle brief
  C. Wire GA4 + GSC into .env
  Gate: First content piece live
  Effort: 2-4 hrs

Total effort: 11-20 hrs

-----------------------------------------------------
TYPE: ai-agent-persona
-----------------------------------------------------

PHASE 1 — Character + Voice Foundation
  Run: /gtm-foundations (abbreviated — skip pricing and sales sections)
  Focus: character backstory, voice bible, narrative arc (where does the persona start,
         where does it go?), onchain event triggers, relationship to parent brand
  Output: outputs/01-character-brief.md
  Gate: Voice bible approved — every line of content flows from this
  Effort: 2-3 hrs

PHASE 2 — Landscape + Audience Research (deep research, run in parallel)
  A. X/Twitter landscape: research-agent → "who are the top AI agent personas on X/Twitter?
     What voice, posting cadence, and narrative arc do they use?"
     Uses: openrouter-research (Grok), apify-mcp
     Output: research/persona-landscape-YYYY-MM-DD.md
  B. Community + audience research: research-agent → "who follows and engages with AI agent
     personas on X/Twitter? What triggers follow, reply, and share?"
     Uses: openrouter-research (Perplexity + Grok)
     Output: research/audience-insights.md
  Gate: Landscape map + top 5 persona models identified and approved
  Effort: 2-3 hrs

PHASE 3 — Design System + Template Library
  A. Visual influences research: designer agent → visual-references skill
     Input: _context/style-guide.md (character aesthetic, persona visual identity)
     Goal: vision board of AI agent persona visuals (avatar styles, text-card aesthetics,
           tone-appropriate references from X/Twitter visual culture)
     Output: _templates/vision-board-[slug].html
  B. Design system lock-in
     Refine _context/style-guide.md with concrete direction from the vision board:
     avatar/profile asset direction, text-card style rules, brand don'ts
  C. Character visual assets: designer agent → canvas-design
     Generate core persona assets:
     - Profile avatar / character illustration (square, 400×400px)
     - Banner / header image (1500×500px, X/Twitter spec)
     - X/Twitter post text-card template (1600×900px): character voice card variant
     Save to: _templates/social-creatives/
  Gate: Gabriel approves vision board + character assets before first content posts
  Effort: 2-3 hrs

PHASE 4 — Content Engine Setup (SOP-17)
  A. Seed first 10 original posts (character voice, opening narrative arc)
  B. Set up daily reply-first cadence (5 targeted replies per day)
  C. Define onchain event triggers and response templates
  Gate: First week of content approved
  Effort: 2-4 hrs

PHASE 5 — Distribution + Cross-Brand Sync
  A. Wire X/Twitter posting schedule
  B. Define cross-brand sync rules (how does this persona reference the parent brand?)
  C. Set up community monitoring via apify-mcp
  Gate: First post live + engagement loop running
  Effort: 1-2 hrs

Total effort: 9-15 hrs

-----------------------------------------------------
TYPE: media-brand  (podcast, newsletter, editorial)
-----------------------------------------------------

PHASE 1 — Strategic Foundation
  Run: /gtm-foundations (focus on content model and monetization, skip product pricing)
  Focus: editorial calendar model, season/episode architecture, distribution strategy,
         sponsorship model vs. subscription, LinkedIn + newsletter growth engine
  Output: outputs/01-phase-1-summary.md
  Gate: Editorial model and monetization path approved
  Effort: 3-4 hrs

PHASE 2 — Audience + Competitive Research (deep research, run in parallel)
  A. ICP research: research-agent → "audience-intelligence-brief for [project name]"
     Focus: who listens/reads, what triggers subscription, what keeps them
     Output: research/audience-insights.md
  B. Competitive content landscape: research-agent → "what podcasts/newsletters exist in this
     space? What topics do they cover? What is the content gap?"
     Uses: openrouter-research (Perplexity + Gemini), apify-mcp
     Output: research/competitive-landscape-content-YYYY-MM-DD.md
  Gate: Audience brief + content gap map approved
  Effort: 2-4 hrs

PHASE 3 — Design System + Template Library
  A. Visual influences research: designer agent → visual-references skill
     Input: _context/style-guide.md (colors, tone adjectives, editorial aesthetic)
     Goal: vision board of references from podcasts, newsletters, and media brands in the vertical
     Output: _templates/vision-board-[slug].html
  B. Design system lock-in
     Refine _context/style-guide.md with concrete direction from the vision board:
     episode cover art direction, newsletter typography rules, brand don'ts
  C. Media brand templates: designer agent → canvas-design + social-graphics
     Generate base templates for all active channels:
     - Episode cover art (3000×3000px, podcast spec)
     - LinkedIn single image (1200×627px): episode promo + quote-card variants
     - X/Twitter post image (1600×900px): episode clip / data-card variant
     - Newsletter header (600×200px)
     - Social clip overlay template (1080×1080px) for audiogram / quote pulls
     Save to: _templates/social-creatives/
  D. Presentation template: branded-deck skill (if pitch decks or sponsor decks are needed)
     Output: _templates/decks/[slug]-media-deck-template.pptx
  Gate: Gabriel approves vision board + full template set before first episode production
  Effort: 2-4 hrs

PHASE 4 — SEO + Content Baseline
  A. SEO onboarding: seo-agent → "SEO onboarding"
  B. Keyword calendar: seo-agent → "keyword-calendar"
     Focus: episode topic keywords + newsletter content pillars
  Gate: Pillar structure and first 3-month calendar approved
  Effort: 2-3 hrs

PHASE 5 — Content Engine Activation (SOP-16)
  A. Approve cycle brief
  B. First episode → blog → social clips → newsletter (SOP-16 sequence)
  C. Wire GA4 + newsletter platform analytics
  Gate: First episode published and repurposed
  Effort: 3-5 hrs

Total effort: 12-20 hrs

-----------------------------------------------------
TYPE: publisher  (SEO-first content site)
-----------------------------------------------------

PHASE 1 — SEO Foundation (primary phase — this type is SEO-first)
  A. SEO onboarding: seo-agent → "SEO onboarding" (seo-aeo-onboarding skill)
     Output: seo/onboarding/YYYY-MM-DD-technical-audit.md + 5 output files
  B. Keyword calendar: seo-agent → "keyword-calendar" (runs automatically after onboarding)
     Output: seo/content-calendar-YYYY-MM-DD.md
  Gate: Pillar architecture approved + first 3-month calendar approved
  Effort: 3-5 hrs

PHASE 2 — Audience + Competitive Research (deep research, run in parallel)
  A. ICP research: research-agent → "audience-intelligence-brief for [project name]"
     Output: research/audience-insights.md
  B. Competitor content audit: research-agent → "top 5 content competitors for [topic] —
     what keywords do they rank for, what topics are underserved?"
     Uses: firecrawl, openrouter-research (Perplexity)
     Output: research/competitive-landscape-content-YYYY-MM-DD.md
  Gate: Competitive content gap approved
  Effort: 2-3 hrs

PHASE 3 — Design System + Template Library
  A. Visual influences research: designer agent → visual-references skill
     Input: _context/style-guide.md (colors, tone adjectives, editorial direction)
     Goal: vision board of SEO content sites and editorial brands in the vertical
     Output: _templates/vision-board-[slug].html
  B. Design system lock-in
     Refine _context/style-guide.md with concrete direction from the vision board:
     blog hero image direction, featured image treatment, brand don'ts
  C. Content site templates: designer agent → canvas-design + social-graphics
     Generate base templates for all active channels:
     - Blog hero / featured image (1200×630px): article type variants (how-to, list, data)
     - LinkedIn single image (1200×627px): article promo card
     - X/Twitter post image (1600×900px): article share card
     - Newsletter header (600×200px) — only if newsletter is a channel
     Save to: _templates/social-creatives/
  Gate: Gabriel approves vision board + template set before first article production
  Effort: 2-3 hrs

PHASE 4 — Content Engine Activation
  A. Approve cycle brief (SEO-driven, from keyword calendar)
  B. First 3 articles per cycle brief: content-writer → seo-agent brief → blog-writer
     Note: 5-article cluster threshold — publish 5 articles on the same pillar before
     expecting AI citation rate to jump (Digital Applied 6.8M-citation data)
  C. Wire GA4 + GSC
  Gate: First article live + indexed
  Effort: 3-5 hrs per article batch

Total effort: 10-16 hrs (excluding article writing time)

-----------------------------------------------------

NOTE: After reading the project type from onboarding.md, include ONLY the matching track
in the post-init checklist output. Do not print all tracks — print only the one that applies.
```

---

## ERROR HANDLING

- If any file write fails, report the error and continue with remaining steps. List all failed writes at the end.
- If `onboarding.md` has a field that is ambiguous (half-filled, placeholder text like "fill this in"), treat it as empty and include it in the soft warnings list.
- Never delete existing files. If a file already exists (e.g., `_context/brand-voice.md`), overwrite it but log that it was overwritten: "OVERWRITE: _context/brand-voice.md (previous version replaced)"
- If the campaigns/[slug]/ folder already exists, do not overwrite the brief.md — append a note instead.

---

## USAGE

```
/init-project
```

Run from the project root directory after filling out `onboarding.md`.