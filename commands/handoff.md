---
description: Manage cross-agent handoff packets in _handoffs/
---

# /handoff — cross-agent handoff threads

Manage `_handoffs/` packets: the vendor-neutral thread system between Claude
Code, Codex, and any future agent. The packet IS the thread — no index file,
no watcher. Convention: `_handoffs/README.md`.

Behavior depends on the argument:

- **No argument** → LIST mode
- **`done <file-or-slug>`** → CLOSE mode
- **Anything else** (slug or short description) → CREATE mode

---

## LIST mode (`/handoff`)

1. Find open packets:
   ```bash
   grep -l '^\*\*Status:\*\* OPEN' _handoffs/[0-9]*.md 2>/dev/null
   ```
2. Also find picked-up packets older than 3 days (stale — likely stalled):
   ```bash
   grep -l '^\*\*Status:\*\* PICKED UP' _handoffs/[0-9]*.md 2>/dev/null
   ```
   For each hit, compare the `**Date:**` line (and any dated appended updates)
   against today; flag if the most recent date is more than 3 days old.
3. For each OPEN packet, read only its header block and `## Next step`
   section, then print a table:

   | File | Title | From → To | Date | Next step |
   |---|---|---|---|---|

   Use the first line under `## Next step` as the summary.
4. If nothing is OPEN and nothing is stale, print exactly:
   `No open handoffs.`
5. Read-only mode: modify nothing, log nothing.

## CREATE mode (`/handoff <slug or description>`)

You are the OUTGOING agent writing the packet at the moment of handoff, from
the CURRENT session's context. This is the whole point — the incoming agent
has zero access to this conversation.

1. Derive a kebab-case slug from the argument (short, descriptive). File:
   `_handoffs/YYYY-MM-DD-<slug>.md` (today's date). If the file already
   exists, append `-2`, `-3`, ...
2. Copy the structure of `_handoffs/_TEMPLATE.md` and fill EVERY section from
   this session's actual context — never leave template placeholders:
   - **Status:** OPEN. **From:** claude-code. **To:** the intended next agent
     (codex / claude-code / any). **Project:** which brand/project this is.
   - **Goal** — the overall task and what "done" looks like.
   - **Current state** — what is completed AND VERIFIED vs. untouched. Only
     claim things you can point to evidence for in this session.
   - **Decisions made (and why)** — the most valuable section. Each decision
     with its reason, so the next agent doesn't relitigate.
   - **Files touched / to read** — repo-relative paths with one-line notes.
   - **Context to load** — which `_context/` files, specs, or briefs.
   - **Next step** — numbered, concrete first actions.
   - **Boundaries** — what the incoming agent must NOT do (board writes, VPS,
     publishing, .env edits). If handing to Codex, always restate: no task
     board or `social/*.json` writes, no VPS, no publishing pipes.
3. Self-containment check before saving: no references to "the chat above",
   "as discussed", session IDs, or secret values (env var NAMES only).
4. Log to the fleet dashboard:
   ```bash
   echo "[$(date '+%H:%M')] ${AGENT_ROLE:-main} ✅ DONE - handoff packet created: <slug>" >> ~/claude-fleet/dashboard.log
   ```
5. End by printing the kickoff line for the other agent, verbatim:

   > Read AGENTS.md, then continue from `_handoffs/<file>.md`

## CLOSE mode (`/handoff done <file-or-slug>`)

1. Resolve the argument to a file in `_handoffs/` (match by exact filename or
   by slug substring; if ambiguous, list the matches and ask).
2. Edit the packet:
   - Flip the header to `**Status:** DONE`.
   - Append at the bottom a dated closing note:
     `### YYYY-MM-DD — closed by <agent>` plus one line on the final outcome.
3. Log to the fleet dashboard:
   ```bash
   echo "[$(date '+%H:%M')] ${AGENT_ROLE:-main} ✅ DONE - handoff closed: <file>" >> ~/claude-fleet/dashboard.log
   ```

## Rules (all modes)

- Never touch task boards, `social/tasks.json`, `social/deliverables.json`,
  the VPS, or any publishing pipe from this command.
- Packets append, never rewrite history: incoming agents add dated sections
  below the `<!-- Incoming agents -->` marker and update only the Status line.
- No secrets in packets. Env var names only.
- DONE packets older than ~30 days may be moved to `_handoffs/archive/`.