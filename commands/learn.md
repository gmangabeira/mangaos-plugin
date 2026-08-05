---
description: Synthesize conversation insights and save to long-term agent memory
---

# /learn — Conversation Memory Synthesis

Scan this conversation for insights worth keeping in long-term memory. Save only what would genuinely help a future Claude work better with Gabriel — things that are non-obvious, hard to re-derive, or would prevent a repeated mistake.

**Focus (optional):** $ARGUMENTS  
If $ARGUMENTS is provided, weight the scan toward that topic. Otherwise scan the full conversation.

---

## Step 1 — Read both memory indexes first

1. Read `MEMORY.md` at the project memory path. Know what's already saved before writing anything.
2. Read `~/Documents/Gabriel Mangabeira/shared/decisions.md`, `shared/research.md`, `shared/skills-log.md`, `shared/project-facts.md` — know what's already in cross-project memory. Do not create duplicates in either store.

---

## Step 2 — Scan for signal

Go through the conversation looking for moments that fit one of these patterns:

**Feedback signals (highest priority):**
- Gabriel corrected your approach ("no, not that", "that's wrong", "come on")
- Gabriel confirmed a non-obvious choice worked ("nice", "exactly", "perfect")
- A repeated mistake — something you got wrong that you'd get wrong again without the memory
- A constraint or preference that isn't visible in the codebase

**Project signals:**
- A decision was made that future sessions need to know about (architecture, naming, strategy)
- A new fact about how the system works that isn't in CLAUDE.md or derivable from code
- A port, path, or configuration detail that will matter again

**Reference signals:**
- A new external resource was identified (URL, tool, account, spreadsheet)
- A location was established for something ("X lives in Y")

**User signals:**
- Something new learned about how Gabriel likes to work or what he cares about

---

## Step 3 — Apply the hard filter

**Do NOT save:**
- Code patterns or architecture already visible in the files
- "We built X" or "we fixed Y" — the fix is in the code, not in memory
- Debugging steps or error messages
- Task lists, in-progress work, or anything session-specific
- Anything already covered in CLAUDE.md or an existing memory file
- Anything a future Claude could figure out in 10 seconds by reading the relevant file

**The bar:** Would a future Claude working on a completely different task still benefit from knowing this? If not, skip it.

---

## Step 4 — Classify: cross-project or project-scoped?

For each insight that passed the filter, ask: **does this apply to 2+ projects in Gabriel's portfolio?**

- **Cross-project** → write to `~/Documents/Gabriel Mangabeira/shared/` (see routing below). Do NOT also write to project memory — avoid duplication.
- **Project-scoped** → write to the project memory directory (Step 5).

**Routing to shared/:**

| Write here | When the insight is about... |
|---|---|
| `shared/decisions.md` | Architectural decisions, portfolio-wide conventions, "from now on" rules, tool standards, constraints that apply across projects |
| `shared/research.md` | Research verdicts applicable beyond this project, tool/framework evaluations, market findings that shape cross-project strategy |
| `shared/skills-log.md` | New skills created, skill propagation status, project-specific vs. shareable classification |
| `shared/project-facts.md` | Project status changes, new ports/URLs/paths, new projects added or archived |

Append to the relevant file. Follow the format already in each file (date prefix, one-paragraph entry). Update `shared/README.md` only if a structural change is needed.

---

## Step 5 — Write project-scoped memories

For each project-scoped insight:

1. Choose the right type: `feedback`, `project`, `reference`, or `user`
2. Pick a filename that describes the rule/fact, not the conversation (e.g., `feedback_one_server_per_project.md`, not `feedback_2026-05-02.md`)
3. Check if an existing memory file covers the same ground — update it instead of creating a new one
4. Write the file to the memory directory with proper frontmatter:

```
---
name: [short descriptive name]
description: [one-line hook — what a future Claude needs to know at a glance]
type: [feedback | project | reference | user]
---

[For feedback: lead with the rule, then **Why:** (the reason/incident), then **How to apply:** (when this kicks in)]
[For project: lead with the fact/decision, then **Why:** (motivation), then **How to apply:** (how this shapes future suggestions)]
[For reference: what it is, where it lives, when to use it]
[For user: who Gabriel is in this context, what to do differently because of it]
```

5. Add a one-line entry to `MEMORY.md` under the existing list (never overwrite the index, only append or update existing lines)

### Step 5b — Prune to budget (required after every append)

After appending, check if MEMORY.md exceeds the **18,432-byte / ~80-line** budget:

```bash
wc -c MEMORY.md   # must be ≤ 18432
```

**If over budget**, demote the oldest non-protected entry to COLD tier:

1. Identify the first entry in MEMORY.md that is **outside** the `<!-- PROTECTED -->` / `<!-- END PROTECTED -->` block. "Oldest" = lowest line number among unprotected entries.
2. Append a line to `MEMORY-archive.md`:
   `- YYYY-MM-DD archived | auto-demoted | [original one-liner] [topic-file.md]`
3. Remove that line from MEMORY.md.
4. Repeat until `wc -c MEMORY.md` ≤ 18,432.

**Never demote** entries between `<!-- PROTECTED -->` and `<!-- END PROTECTED -->`.

---

## Step 6 — Report

Output a clean summary:

**Saved to shared/ ([N]):**
- `shared/decisions.md` — [one line: the insight]
- ...

**Saved to project memory ([N]):**
- `filename.md` — [one line: the insight]
- ...

**Discarded (with reason):**
- [insight] — [why it didn't pass: already in memory / derivable from code / ephemeral / etc.]

If nothing new passed the filter, say so directly: "Nothing new worth saving — all insights are already in memory or derivable from the codebase."