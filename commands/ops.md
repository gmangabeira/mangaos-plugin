---
description: Run the marketing project manager ops snapshot
---

Run the marketing-project-manager skill. Generate a full ops snapshot for today.

Read `social/tasks.json` and `campaigns/campaigns.json`. Output all five sections in order:
1. Active Campaigns (with stale gate flags)
2. Needs Your Approval Now (tasks + campaign gates, ranked by age)
3. Ships This Week (Monday to Sunday of current week, by due date)
4. Blocked (tasks where status is blocked)
5. Quick Health (computed stats)

Do not ask clarifying questions. Read the files and output the snapshot immediately.