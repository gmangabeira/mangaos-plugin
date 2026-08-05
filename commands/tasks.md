---
description: View or modify the marketing task board
---

Read `social/tasks.json` and display or modify the task board.

## Commands
- **Show tasks**: Display all tasks grouped by status (todo, inprogress, review, done)
- **Add task**: Add a new task with title, category, priority, and optional fields below
- **Move task**: Change a task's status (e.g., move "pub-01" to inprogress). Always set `statusChangedDate` to today when status changes.
- **Complete task**: Mark a task as done with today's date
- **Filter**: Show tasks by category, channel, campaign, or approvalStatus

## Task file
The source of truth is `social/tasks.json` (schema v2.0). Always read it before making changes. After any modification, write the updated JSON back and update `lastUpdated`.

The dashboard at `http://localhost:3333` polls tasks.json every 3 seconds via `task-server.js`, so changes you make here appear in the browser automatically.

## Required fields
- `id` — format: category-prefix + number (e.g., "pub-04", "infra-08")
- `title` — human-readable task name
- `category` — see categories below
- `status` — todo | inprogress | review | done | blocked
- `priority` — high | medium | low
- `created` — "YYYY-MM-DD"
- `statusChangedDate` — "YYYY-MM-DD" — set to today whenever status changes

## Optional fields (schema v2.0)
- `campaign` — campaign slug matching `campaigns/campaigns.json` (e.g., "post-launch-death-pattern-q2-2026")
- `channel` — "linkedin" | "x" | "newsletter" | "blog" | "landing-page" | "ads" | "seo" | "research" | "ops"
- `dueDate` — "YYYY-MM-DD" — deadline for ships-this-week tracking
- `approvalStatus` — "not-needed" | "pending" | "approved" | "rejected" — set to "pending" when moving to review
- `sop` — SOP reference (e.g., "SOP-05")
- `cluster` — content cluster slug (can coexist with campaign)
- `notes` — internal notes, instructions, context
- `url` — published post URL
- `completed` — "YYYY-MM-DD" set when status → done
- `links` — array of {type, url} external references
- `attachments` — array of {name, type, size, thumb} file metadata
- `body` — full post content (for content tasks)

## Categories
- `content` — Posts, threads, articles, repurposing
- `x-growth` — X/Twitter growth engine tasks (SOP-09)
- `infrastructure` — Tools, SOPs, skills, automation setup
- `research` — Market intel, competitive scans, protocol analysis

## Starting the task board
Run: `node task-server.js`
Then open: `http://localhost:3333`
Campaign View toggle shows tasks grouped by campaign.
"Pending Review" stat shows count of tasks with approvalStatus: pending.

## Visual tools
The daily replies dashboard is at `social/x/x-daily-replies.html`.
The reply targets feed is at `social/x/x-reply-feed.html`.

## Related
For a full project status snapshot (campaigns + approval queue + ships-this-week + blocked), use `/ops` instead.