---
description: Sync skills with vps_runtimes to their VPS containers
---

# /sync-vps-skills

Sync skills that have `vps_runtimes` set in their frontmatter from the `marketing/` workspace (the source of truth) to their target OpenCLAW VPS containers.

## Trigger
Run when: after modifying a skill that has `vps_runtimes: [...]` in its frontmatter, after adding a new skill with `vps_runtimes`, or after running `init-project` on a VPS container.

## Steps

1. Read `_ops/vps-skills-manifest.json` to get the list of skills to sync (and their `vps_runtimes`).
2. For each skill in the manifest:
   a. Confirm the skill's SKILL.md has the matching `vps_runtimes` field.
   b. For each runtime in `vps_runtimes`, rsync the skill folder to that container's **default workspace skills dir** (see VPS paths below).
   c. **chown the synced folder to `1000:1000`** (the runtime runs as uid 1000 / `ubuntu`; root-owned files are unreadable to it — this step is mandatory, not optional).
   d. Verify the files landed (ls the target, confirm `ubuntu` ownership).
   e. Update `sync_status` to `synced` and set `synced_at` to today's ISO date in the manifest.
3. Use the `openclaw-vps` skill (or the SSH key directly) for SSH/rsync operations.
4. After all syncs: write the updated manifest back to `_ops/vps-skills-manifest.json`.
5. Report per runtime: N skills synced to mrzq, N synced to h3mk, and the **pickup caveat** (see below).

## VPS paths (CORRECTED 2026-06-08 — the old `/docker/openclaw-<inst>/workspace/skills` path was wrong and never worked)
- Host: `root@187.77.42.134`
- SSH key: `~/.ssh/hostinger_vps`
- **mrzq** (Maestro + 4 C-suite): `/docker/openclaw-mrzq/data/.openclaw/workspace/skills/`
- **h3mk** (Sasha): `/docker/openclaw-h3mk/data/.openclaw/workspace/skills/`

There are three skill scopes per instance under `data/.openclaw/`: the **default workspace** `workspace/skills/` (shared by all agents in that instance — use this for marketing skills), the instance-**global** `skills/`, and **per-agent** `workspace-<agent>/skills/` (e.g. `workspace-maestro/skills`). Default-workspace placement is the right target for skills meant for every agent.

## Skill discovery (no config edit needed)
Skills are **auto-discovered**: `openclaw.json` has `"skills": { "entries": {} }` and `"nativeSkills": "auto"`, so dropping a correctly-owned folder into the skills dir is enough — there is no registry entry to add. **Pickup caveat:** an already-running container loads skills at startup, so a newly-synced skill is typically picked up on the **next container restart**. rsync + chown only proves the files are present and readable; it does NOT prove the running agents have loaded the skill. Do not claim a skill is "live on the VPS" until a restart or an observed agent run confirms it. Do not restart live containers without explicit approval (it interrupts Maestro/C-suite and Sasha's cadence).

## Example (use `-L` to follow symlinks, `--delete` scoped to the skill subfolder, then chown)
```bash
KEY="$HOME/.ssh/hostinger_vps"
SRC="/Users/gabrielmangabeira/Documents/Gabriel Mangabeira/marketing/.claude/skills/dune-analytics/"
DST="/docker/openclaw-mrzq/data/.openclaw/workspace/skills/dune-analytics/"

# -L dereferences symlinked skill folders (e.g. dune, sim) so their real content syncs, not a broken link.
# Trailing slashes on both sides scope --delete to this one skill folder (safe; never touches sibling skills).
rsync -avL --delete -e "ssh -i $KEY" "$SRC" "root@187.77.42.134:$DST"

# Mandatory: make it readable by the uid-1000 runtime.
ssh -i "$KEY" root@187.77.42.134 "chown -R 1000:1000 ${DST%/}"

# Verify.
ssh -i "$KEY" root@187.77.42.134 "ls -la ${DST}references"
```

## Known backlog (2026-06-08)
The 8 pre-existing manifest entries (`qa-content`, `buffer-publish`, `linkedin-post-writer`, `blog-writer`, `seo-aeo-geo`, `social-graphics`, `web3-twitter-thread-writer`, `ad-creative`) are all `pending` and were never actually synced. (`mangabeira-blog-writer` is deprecated as of 2026-06-18 — use `blog-writer`.) Before syncing them, audit against the VPS's existing skill set (`data/.openclaw/workspace/skills/`) — several likely overlap with skills already present under other names (for example manifest `linkedin-post-writer` vs the VPS's existing `linkedin-writer`). Sync only genuine gaps.