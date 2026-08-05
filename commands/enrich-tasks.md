---
description: Enrich task board fields from linked campaign briefs
---

Run the task board enrichment. Read `social/tasks.json`, `campaigns/campaigns.json`, and each linked campaign's `campaigns/[slug]/brief.md`. Then fill in every inferrable field on every task so the next agent can execute without asking questions.

**Do NOT change:** id, title, status, priority, created, completed, url, attachments, body (if already has real content).
**Only add values to fields that are empty or undefined.**

---

## Step 1 — Load context

1. Read `social/tasks.json`
2. Read `campaigns/campaigns.json`
3. For each active campaign, read `campaigns/[slug]/brief.md`
4. Read `_sop/` index from CLAUDE.md SOP table to know which SOP maps to which workflow

---

## Step 2 — Enrich each task

For every task, fill the following fields if missing:

### channel
- ID prefix `xgrowth-` or category `x-growth` → `"x"`
- ID prefix `linkedin-` → `"linkedin"`
- ID prefix `infra-` or category `infrastructure` → `"ops"`
- ID prefix `research-` or category `research` → `"research"`
- ID prefix `pub-` → `"linkedin"`
- ID prefix `repurpose-`: `"blog"` if title mentions blog/article/mangabeira, `"linkedin"` otherwise
- title mentions "newsletter" → `"newsletter"`
- title mentions "landing page" or "page" → `"landing-page"`
- title mentions "SEO" or "keyword" → `"seo"`
- When uncertain, derive from category.

### campaign
- If task has a `cluster` field, match to campaign id in campaigns.json (find campaign whose id contains the cluster value). Set `campaign` to that campaign's id.
- If task has no cluster but title clearly matches an active campaign's theme, infer and set.

### cluster
- If task has `campaign` set but no `cluster`, derive the cluster from the campaign id (strip the `-q[N]-[YEAR]` suffix).

### sop
Use this mapping. Pick the most specific match:

| Match condition | SOP |
|---|---|
| category `x-growth` | SOP-09 |
| category `research` | SOP-10 |
| channel `newsletter` | SOP-11 |
| channel `seo` or title contains "SEO refresh" | SOP-12 |
| channel `landing-page` or title contains "landing page" | SOP-13 |
| title contains "weekly report" or "performance report" | SOP-07 |
| title contains "monthly" and category `infrastructure` | SOP-08 |
| title contains "weekly intelligence" or "intel" | SOP-01 |
| title contains "repurpose" or "expand" and channel `blog` | SOP-04 |
| title contains "repurpose" or "expand" and channel `linkedin` | SOP-04 |
| title contains "repurpose" and channel `x` | SOP-04 |
| title contains "post" or "publish" and channel `linkedin` | SOP-05 |
| title contains "post" or "thread" and channel `x` | SOP-05 |
| category `content` and no better match above | SOP-03 |
| category `infrastructure` | (leave blank — infra tasks vary) |

### approvalStatus
- status `"review"` → `"pending"`
- status `"done"` → `"approved"`
- all other statuses → `"not-needed"`

### statusChangedDate
- Set to the task's `created` date as a conservative baseline.

### dueDate
- If task has a `campaign`, look up that campaign's `endDate` in campaigns.json. Set `dueDate` to the campaign `endDate`. This gives every campaign task a visible deadline.
- Do not set dueDate on tasks with no campaign.

### notes
- If notes field is empty or just whitespace, write a one-sentence execution note based on the task title, SOP, and campaign context. Keep it under 120 characters. Examples:
  - "Expand X thread into full article. Use linkedin-post-to-article skill. Campaign: The Post-Launch Death Pattern."
  - "Run SOP-10 research sweep. Produce intel brief for next cycle planning."
  - "Weekly report. Pull GA4 + GSC data. Summarize channel performance vs. cycle targets."
- Do not overwrite notes that already contain real content.

---

## Step 3 — Write and report

1. Set `lastUpdated` on the root object to today's date (YYYY-MM-DD).
2. Write the updated JSON back to `social/tasks.json`.
3. Print a table showing: Task ID | Fields filled | Brief summary of what was added.

## Step 4 — Print ready queue

After the enrichment table, print a **Ready to Dispatch** section listing all tasks where:
- `status === "todo"` AND
- `sop` is set (non-empty)

Format:
```
## Ready to Dispatch
/run-task [id] — [title] ([sop] · [channel])
/run-task [id] — [title] ([sop] · [channel])
...
```

If no tasks qualify, print: `Ready to Dispatch: nothing queued.`

This gives Gabriel a copy-pasteable dispatch list every morning without opening the board.