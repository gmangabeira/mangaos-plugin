---
description: Execute the task delegation workflow for a given task ID
---

Execute the task delegation workflow for the task ID provided as the argument to this command.

---

## Step 1 — Load the task

Run this Node.js command to extract the task (tasks.json is too large to Read directly due to base64 thumbnails):

```
node -e "const d=JSON.parse(require('fs').readFileSync('social/tasks.json','utf8')); const t=d.tasks.find(t=>t.id==='TASK_ID'); console.log(JSON.stringify(t,null,2));"
```

Replace `TASK_ID` with the argument passed to this command. If no task is found, stop and tell the user: "Task [ID] not found in tasks.json."

---

## Step 2 — Load campaign brief (if applicable)

If the task has a `campaign` field set, read `campaigns/[campaign-value]/brief.md`. This gives the agent the narrative spine, ICP, CTA, and campaign timeline.

If no `campaign` field, skip this step.

---

## Step 3 — Identify the agent

Use the task's `sop` field and `channel` field to pick the agent:

| SOP | Agent |
|-----|-------|
| SOP-03 | content-writer |
| SOP-04 | content-writer |
| SOP-05 | content-writer |
| SOP-06 | (direct) — engagement loop. Do not delegate. Tell user to run engagement actions manually per SOP-06. |
| SOP-07 | data-analyst |
| SOP-08 | research-agent |
| SOP-09 | (direct) — X growth ops. Do not delegate. Tell user these are manual operational steps per SOP-09. |
| SOP-10 | research-agent |
| SOP-11 | content-writer |
| SOP-12 | data-analyst |
| SOP-13 | designer (Steps 4+). Note: Steps 1-3 (copy outline + Gate 1) must be completed by content-writer first. If body/copy is not already in the task, route to content-writer for Step 1-3 before designer. |
| blank + channel `ops` | (direct) — infrastructure task. Do not delegate. Describe what the task requires and link to the relevant SOP if one exists. |
| blank + channel `linkedin`, `x`, `blog`, or `newsletter` | content-writer |
| blank + channel `research` | research-agent |
| blank + channel `seo` | data-analyst |
| blank + no channel + category `infrastructure` | (direct) |

---

## Step 4 — Update task status

If the task's current `status` is `todo`, update it to `inprogress` and set `statusChangedDate` to today's date (YYYY-MM-DD).

Run this Node.js command to write the update:

```
node -e "
const fs = require('fs');
const path = 'social/tasks.json';
const d = JSON.parse(fs.readFileSync(path, 'utf8'));
const t = d.tasks.find(t => t.id === 'TASK_ID');
if (t && t.status === 'todo') {
  t.status = 'inprogress';
  t.statusChangedDate = 'TODAY_DATE';
  d.lastUpdated = 'TODAY_DATE';
  fs.writeFileSync(path, JSON.stringify(d, null, 2));
  console.log('Status updated: todo → inprogress');
} else {
  console.log('Status unchanged:', t ? t.status : 'task not found');
}
"
```

---

## Step 5 — Load context files

Based on channel, load these context files before building the agent prompt:

| Channel | Files to load |
|---------|---------------|
| linkedin, x, blog, newsletter | `_context/brand-voice.md`, `_context/audience.md` |
| seo | `_context/brand-voice.md`, `_context/audience.md`, `_context/product-info.md` |
| landing-page | `_context/brand-voice.md`, `_context/audience.md`, `_context/product-info.md`, `_context/html-elements.md`, `_context/style-guide.md` |
| research | `_context/audience.md`, `_context/product-info.md` |
| ops | (none required) |

---

## Step 6 — Delegate to the agent

Output one line first: `Delegating [task-id] — [title] → [agent-name]`

Then use the Agent tool to delegate. Build the agent prompt using this structure:

```
You are executing task [TASK_ID] from the MangaOS task board.

**Task:** [title]
**Channel:** [channel]
**SOP:** [sop]
**Campaign:** [campaign name, if applicable]
**Notes:** [task notes field]

**Campaign context** (from brief.md, if applicable):
[paste relevant sections: goal, ICP, narrative spine, CTA]

**Task body** (if present):
[paste task body/attachments content]

**Your job:**
Follow [SOP name and file path] as your primary workflow.
Load brand context from: [list context files].
Produce a finished deliverable — not a draft.
Route the output to [correct folder per CLAUDE.md routing rules].
Stop and flag [NEEDS APPROVAL] at any human judgment gate defined in the SOP.
```

Set `subagent_type` based on the routing table above:
- content-writer → `subagent_type: "content-writer"`
- data-analyst → `subagent_type: "data-analyst"`
- designer → `subagent_type: "designer"`
- research-agent → `subagent_type: "research-agent"`

---

## For direct tasks (no delegation)

If the task routes to `(direct)`, do not use the Agent tool. Instead:

1. Print: `Task [task-id] — [title] is a direct-execution task ([channel]/[sop]).`
2. Describe what the task requires in 2-3 sentences.
3. Link to the relevant SOP.
4. Tell the user what they need to do manually, and what you can help with inline.