---
description: Force-run all MSE ops cadences (daily/weekly/monthly) on demand
---

# /startup — MSE Ops Manual Trigger

Force-check all three MSE cadences right now, regardless of whether they already ran today/this week/this month. Use this when you want a fresh ops run on demand.

---

## What this does

Runs all three MSE scheduled checks in sequence:

1. **Daily preflight** — MCP connections, pending approvals staleness, active cycle brief, broken file refs, fleet log writability. Overwrites `_ops/preflight-report.md`.

2. **Weekly maintenance** — Skill/skills-reference sync, context file staleness, agent memory audit, BLOCKED fleet entries, SOP reference check. Overwrites `_ops/weekly-report.md`. Applies autonomous fixes and appends to `_ops/infra-log.md`.

3. **Monthly infra review** — Agent defs vs CLAUDE.md coverage, skills vs SOPs matrix, MCP inventory, pending approvals backlog, changelog cross-reference, infra log health. Overwrites `_ops/monthly-report.md`. Read-only.

---

## Instructions

Run each in sequence. Do not skip any. For each one:

1. Delegate to the `marketing-systems-engineer` agent with the appropriate trigger keyword
2. Wait for completion
3. Print a one-line summary of the result

**Step 1 — Daily preflight:**
Delegate to `marketing-systems-engineer` agent: `RUN_DAILY_PREFLIGHT`

**Step 2 — Weekly maintenance:**
Delegate to `marketing-systems-engineer` agent: `RUN_WEEKLY_MAINTENANCE`

**Step 3 — Monthly infra review:**
Delegate to `marketing-systems-engineer` agent: `RUN_MONTHLY_INFRA_REVIEW`

After all three complete, print a brief summary:
- Total flags from preflight
- Autonomous fixes applied in weekly
- Items queued in pending-approvals.md
- Any escalations from monthly