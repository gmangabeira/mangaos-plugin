---
description: Execute a Content Engine pipeline stage for a given task
---

Execute a Content Engine pipeline stage for a given task. Called by `/watch-queue` when a CE item is dispatched.

Arguments: `[taskId] [stage]`

---

## Step 1 — Load the task

```
node -e "const d=JSON.parse(require('fs').readFileSync('social/tasks.json','utf8')); const t=d.tasks.find(t=>t.id==='TASK_ID'); console.log(JSON.stringify(t,null,2));"
```

Replace `TASK_ID`. If not found, stop: "Task [ID] not found."

## Step 2 — Load campaign brief (if task has `campaign` field)

Read `campaigns/[campaign]/brief.md` for narrative spine, ICP, CTA, timeline.

## Step 3 — Stage routing

| Stage | Agent | Action |
|---|---|---|
| `research` | research-agent | Research the topic. Save output to `seo/briefs/[slug]-research.md`. Update task `artifacts.research`. |
| `outline` | content-writer | Generate content outline. Save to `seo/briefs/[slug]-outline.md`. Update `artifacts.outline`. Set `approvalStatus: "pending"` — wait for human approval before proceeding. |
| `writing` | content-writer | Write full draft from outline (must exist at `artifacts.outline`). Save blog draft to `seo/[slug].md`, LinkedIn to `social/linkedin/li-post-[date]-[slug].md`, X thread to `social/x/x-thread-[date]-[slug].md` per channels array. Update `artifacts.draft_blog`, `artifacts.draft_linkedin`, `artifacts.draft_x`. |
| `design` | designer | Generate companion image(s) for each channel in `channels[]`. Save to `social/linkedin/li-img-[date]-[slug].png` or `social/x/x-img-[date]-[slug].png`. Update `artifacts.design[]`. |
| `editing` | content-writer | Self-edit pass against brand-voice.md checklist. Output revised file at same path (overwrite). Set `approvalStatus: "pending"`. |
| `distribution` | content-writer | Write distribution variants from the published piece: LinkedIn hook variants, X cross-post thread, newsletter excerpt. Save alongside the canonical. Update `artifacts.published[]`. |

## Step 4 — Update task status to `inprogress`

```
node -e "
const fs=require('fs');
const p='social/tasks.json';
const d=JSON.parse(fs.readFileSync(p,'utf8'));
const t=d.tasks.find(t=>t.id==='TASK_ID');
if(t){ t.status='inprogress'; t.statusChangedDate=new Date().toISOString().slice(0,10); d.lastUpdated=t.statusChangedDate; fs.writeFileSync(p,JSON.stringify(d,null,2)); console.log('ok'); }
"
```

## Step 5 — Build agent prompt

Print: `[run-ce-stage] TASK_ID — STAGE → AGENT`

Then use the Agent tool to delegate. Structure the prompt:

```
You are executing Content Engine stage "[STAGE]" for task [TASK_ID] in the MangaOS pipeline.

**Task:** [title]
**Stage:** [stage]
**Channels:** [channels array]
**Campaign:** [campaign name, if any]
**Notes:** [task notes]

**Campaign context** (from brief.md):
[paste: goal, ICP, CTA, narrative spine]

**Artifacts already produced** (read these before starting):
- Research: [artifacts.research path, if exists]
- Outline: [artifacts.outline path, if exists]
- Previous draft: [artifacts.draft_blog path, if exists]

**Your job:**
1. Load brand context from: _context/brand-voice.md, _context/audience.md[, _context/product-info.md if writing or distribution].
2. Execute the [STAGE] stage per the routing table in .claude/commands/run-ce-stage.md.
3. Save output to the correct path (see routing table).
4. After saving, update social/tasks.json: set artifacts.[key] to the output file path and advance pipelineStage if this stage does NOT require human approval.
5. If stage is "outline" or "editing": do NOT advance pipelineStage. Set approvalStatus: "pending" and stop. The human will approve from the dashboard.
6. Route all outputs correctly per CLAUDE.md routing rules.
7. Flag [NEEDS APPROVAL] if any judgment call requires human input.
```

Set `subagent_type`:
- `research-agent` for research stage
- `content-writer` for outline, writing, editing, distribution stages
- `designer` for design stage

## Step 6 — After agent completes

Update task status back to `todo` if stage sets `approvalStatus: "pending"`, or to `done` if stage is complete and does not require approval.

If the agent wrote an artifact path, confirm the file exists:
```
node -e "const fs=require('fs'); console.log(fs.existsSync('PATH') ? 'exists' : 'MISSING');"
```

Report: `[run-ce-stage] DONE — TASK_ID [stage] → [artifact path]`