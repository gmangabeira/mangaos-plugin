---
description: Check the dispatch queue and execute pending tasks
---

Check `social/dispatch-queue.json` for pending tasks and execute them.

## Step -1 — Acquire lock

Write a lockfile to prevent double-execution across parallel sessions:

```
node -e "
const fs=require('fs');
const lockFile='social/dispatch-queue.lock';
if (fs.existsSync(lockFile)) {
  const age = Date.now() - fs.statSync(lockFile).mtimeMs;
  if (age < 300000) { console.log('LOCKED'); process.exit(0); }
  // Stale lock (>5min) — override it
}
fs.writeFileSync(lockFile, String(Date.now()));
console.log('OK');
"
```

If output is `LOCKED`: print `[watch-queue] Queue locked by another session — skipping this tick.` and stop.

Remove the lock at the end of each full tick (Step 3e or when queue is empty):
```
node -e "try{require('fs').unlinkSync('social/dispatch-queue.lock')}catch(e){}"
```

## Step 0 — Heartbeat

Fire a tick to task-server FIRST so the dashboard knows the worker is alive. Non-blocking; ignore failures (task-server may be offline).

```
curl -s -X POST http://localhost:3333/api/queue/tick -H 'Content-Type: application/json' -d '{"consumed":false}' >/dev/null 2>&1 || true
```

When an item is actually consumed in step 3c, fire a second tick with `consumed:true` so the dashboard's "last consumed" timestamp updates:

```
curl -s -X POST http://localhost:3333/api/queue/tick -H 'Content-Type: application/json' -d '{"consumed":true}' >/dev/null 2>&1 || true
```

## Step 0a — Pull remote dispatches from the OpenCLAW bridge

The OpenCLAW dispatch-to-marketing bridge (running on the VPS) writes dispatches directly into this repo on GitHub. Before reading the queue, sync down so we see remote-written entries. Gentle: if the local working tree can't fast-forward (uncommitted changes that touch the same files), continue with current local state — the next tick will retry.

```
git fetch origin main 2>/dev/null || true
git merge --ff-only origin/main 2>/dev/null || echo "[watch-queue] Note: could not ff-merge — proceeding with local state"
```

## Step 1 — Read the queue

Read the queue file and normalize to a flat item list using the compat reader:

```
node -e "
const fs=require('fs');
const raw=fs.readFileSync('social/dispatch-queue.json','utf8');
let parsed; try{parsed=JSON.parse(raw)}catch(e){parsed={}}
let items=[];
if(parsed.schema==='v2'){
  items=Object.values(parsed.queues||{}).flat();
} else {
  items=Array.isArray(parsed.queue)?parsed.queue:[];
}
console.log(JSON.stringify({schema:parsed.schema||'v1',items,deadLetterCount:(parsed.deadLetter||[]).length}));
"
```

If `items` is empty and `deadLetterCount > 0`, print `[watch-queue] Queue empty. Dead-letter has ${N} items — review at http://localhost:3333 Fleet tab.` then remove the lock and stop.

If `items` is empty: remove the lock and print nothing.

## Step 2 — If queue is empty

Print nothing. The loop will check again on the next interval.

## Step 3 — If queue has items

For each item in `queue`, in order:

### 3a — Claim the item (atomic read-modify-write to prevent double-execution)

```
node -e "
const fs=require('fs');
const f='social/dispatch-queue.json';
const raw=fs.readFileSync(f,'utf8');
let d;try{d=JSON.parse(raw)}catch(e){d={queue:[]}}
let item=null;
if(d.schema==='v2'){
  // Find first item across all enabled runtimes, prefer mrzq
  const order=['mrzq','h3mk','hermes'];
  for(const rt of order){
    if(d.runtimes&&d.runtimes[rt]&&d.runtimes[rt].enabled===false)continue;
    if(d.queues[rt]&&d.queues[rt].length>0){item=d.queues[rt].shift();break;}
  }
} else {
  item=d.queue&&d.queue.shift();
}
if(item)fs.writeFileSync(f,JSON.stringify(d,null,2));
console.log(item?JSON.stringify(item):'empty');
"
```

If output is `empty`: release the lock, then stop here.
```
node -e "try{require('fs').unlinkSync('social/dispatch-queue.lock')}catch(e){}"
```

### 3b — Identify the item type

**Pre-flight QA gate (check this FIRST, before any action type matching):**

If the claimed item has a `taskId` field (i.e. it is a CE item or any task-linked dispatch), look up the task in `social/tasks.json` by `taskId`. If `task.pipelineStage === 'editing'` AND `task.qaStatus` is set AND `task.qaStatus !== 'passed'` AND `task.qaStatus !== 'not-run'`, skip this item. Do NOT dispatch it. Log:

```
[watch-queue] Skipping ${item.taskId}: editing stage, qaStatus=${task.qaStatus} — QA gate not cleared
```

Before moving to the next item, write the item back into `d.queues[rt]` (append to the end — do NOT put it back at position 0, to avoid a tight loop) and write the queue file. Then proceed to the next item.

This ensures a QA-blocked task is re-queued for the next poll tick rather than silently dropped. The task will keep cycling until QA is cleared.

```
node -e "
const fs=require('fs');
const f='social/dispatch-queue.json';
const raw=fs.readFileSync(f,'utf8');
let d;try{d=JSON.parse(raw)}catch(e){d={queue:[]}}
const item=JSON.parse(process.argv[1]);
if(d.schema==='v2'){
  const rt=item.runtime||'mrzq';
  if(!d.queues[rt])d.queues[rt]=[];
  d.queues[rt].push(item); // append to END, not position 0
} else {
  if(!d.queue)d.queue=[];
  d.queue.push(item);
}
fs.writeFileSync(f,JSON.stringify(d,null,2));
" -- '<item JSON>'
```

Then proceed to the next item in the queue (loop continues — do NOT dead-letter this item; it is a gate block, not a failure). Release the lock normally when the loop exhausts all items.

If the task is not found in `social/tasks.json` (orphaned dispatch), allow it to proceed normally.

---

- **`type: 'blog-refresh'`** (research → brief → draft → QA → Google Doc): Execute `/blog-refresh-run [taskId] [slug] [track]`
- **`type: 'blog-publish'`** (translate → CMS publish → IndexNow → done): Execute `/blog-publish-run [taskId] [slug]`
- **CE item** (has `taskId` + `stage` fields, no `type`): Execute `/run-ce-stage [taskId] [stage]`
- **Legacy ops item** (has only `id` field): Execute `/run-task [id]`

- **`type: 'social-publish-linkedin'`** (queue a LinkedIn post via Buffer): Read `item.text` from the claimed JSON item. **Never interpolate item fields directly into shell argument strings.** Write the text to a temp file first using Node (passing the raw string value as a Node process argument, not shell-interpolated):
  ```bash
  node -e "require('fs').writeFileSync('/tmp/dispatch-post-text.txt', process.argv[1])" -- "<item.text_raw_value>"
  ```
  Then run:
  ```bash
  node .claude/skills/buffer-publish/scripts/post.js \
    --account MANGABEIRA --channel linkedin \
    --body-file /tmp/dispatch-post-text.txt
  ```
  If the item has an `account` field, use that value instead of `MANGABEIRA`. On SUCCESS, set `status: "done"` and include the returned post id in `artifacts`. On FAILURE (non-zero exit from the script), do NOT set `status: "done"`. Instead, treat this item as a failed execution and proceed with Step 3f dead-letter logic (increment `item.attempts`, dead-letter if >=3, re-queue with backoff if <3).

- **`type: 'social-publish-x'`** (queue an X post via Buffer): Read `item.text` from the claimed JSON item. **Never interpolate item fields directly into shell argument strings.** Write the text to a temp file first using Node (passing the raw string value as a Node process argument, not shell-interpolated):
  ```bash
  node -e "require('fs').writeFileSync('/tmp/dispatch-post-text.txt', process.argv[1])" -- "<item.text_raw_value>"
  ```
  Then run:
  ```bash
  node .claude/skills/buffer-publish/scripts/tweet.js \
    --account MANGABEIRA \
    --body-file /tmp/dispatch-post-text.txt
  ```
  If the item has an `account` field, use that value instead of `MANGABEIRA`. If the item has a `mediaUrl` field, add `--media-url` followed by the URL value (write to a temp var, do not shell-interpolate into a quoted string with untrusted content). On SUCCESS, set `status: "done"` and include the returned post id in `artifacts`. Note: Buffer does not support X reply threading — single posts only. For reply threads, the item must use the termux-bridge path (Sasha only). On FAILURE (non-zero exit from the script), do NOT set `status: "done"`. Instead, treat this item as a failed execution and proceed with Step 3f dead-letter logic (increment `item.attempts`, dead-letter if >=3, re-queue with backoff if <3).

- **`type: 'newsletter-draft'`** (create a beehiiv draft): Read `item.title`, `item.body`, and `item.bodyFile` from the claimed JSON item. **Never interpolate item fields directly into shell argument strings.** Write `item.title` to a temp file first using Node (passing the raw string value as a Node process argument, not shell-interpolated):
  ```bash
  node -e "require('fs').writeFileSync('/tmp/dispatch-newsletter-title.txt', process.argv[1])" -- "<item.title_raw_value>"
  ```
  If `item.body` is provided, write it to a temp file first using Node (passing the raw string value as a Node process argument, not shell-interpolated):
  ```bash
  node -e "require('fs').writeFileSync('/tmp/dispatch-newsletter-body.txt', process.argv[1])" -- "<item.body_raw_value>"
  ```
  Then run:
  ```bash
  node .claude/skills/beehiiv-draft/scripts/create-draft.js \
    --title-file /tmp/dispatch-newsletter-title.txt \
    --body-file /tmp/dispatch-newsletter-body.txt
  ```
  If the item has a `bodyFile` field instead of `body`, use `--body-file "<item.bodyFile>"` directly (this is a file path, not user content). If the item has a `subtitle` field, write it to a temp file and pass `--subtitle-file`:
  ```bash
  node -e "require('fs').writeFileSync('/tmp/dispatch-newsletter-subtitle.txt', process.argv[1])" -- "<item.subtitle_raw_value>"
  ```
  Then add `--subtitle-file /tmp/dispatch-newsletter-subtitle.txt` to the `create-draft.js` invocation. On SUCCESS, set `status: "done"` and include the returned `{ id, web_url }` in `artifacts`. On FAILURE (non-zero exit from the script), do NOT set `status: "done"`. Instead, treat this item as a failed execution and proceed with Step 3f dead-letter logic (increment `item.attempts`, dead-letter if >=3, re-queue with backoff if <3).

- **`type: 'sasha-post-x'`** [h3mk runtime only] — Post an original tweet to Sasha's X account via Buffer.

  **ARCHITECTURE CONSTRAINT — DO NOT DEVIATE:** Sasha's original posts go through Buffer. Same path as mrzq `social-publish-x`. X API (`tweet.js`) is NOT used — cost constraint ($200+/mo). Replies go through the phone/ADB termux-bridge (`sasha-reply-x`), not this case.

  **Never interpolate `item.text` directly into shell strings.** Write the text to a temp file first using Node:
  ```bash
  node -e "require('fs').writeFileSync('/tmp/dispatch-post-text.txt', process.argv[1])" -- "<item.text_raw_value>"
  ```
  Then run:
  ```bash
  node .claude/skills/buffer-publish/scripts/tweet.js \
    --account SASHA_COIN \
    --body-file /tmp/dispatch-post-text.txt
  ```
  Use `item.account` value if present, otherwise default to `SASHA_COIN`. On SUCCESS: set `status: "done"`, log the returned Buffer post ID in `artifacts.published`. On FAILURE (non-zero exit): apply Step 3f dead-letter logic.

- **`type: 'sasha-reply-x'`** [h3mk runtime only] — Post a reply to a specific tweet via Sasha's Android phone (ADB, real device fingerprint via termux-bridge).

  **ARCHITECTURE CONSTRAINT — DO NOT DEVIATE:** Sasha's replies go through the phone/ADB via `x-engage.js` inside the h3mk container. Buffer cannot model reply threading. X API is not used (cost). Posts go through Buffer (`sasha-post-x`).

  Payload fields: `item.text` (reply text) and `item.replyTo` (full tweet URL, e.g. `https://x.com/user/status/123`). If `item.replyTo` is empty, dead-letter immediately (configuration error).

  **Never interpolate `item.text` or `item.replyTo` directly into shell strings.** Write the full dispatch payload to a temp file via Node first:
  ```bash
  node -e "
  const item = JSON.parse(process.argv[1]);
  const payload = {type: 'sasha-reply-x', payload: {text: item.text || '', replyTo: item.replyTo || ''}};
  require('fs').writeFileSync('/tmp/h3mk-dispatch.json', JSON.stringify(payload));
  " -- '<item JSON>'
  ```

  Then SSH to the VPS and pipe to the dispatch script:
  ```bash
  DISPATCH_RESULT=$(ssh -i ~/.ssh/hostinger_vps \
    -o ConnectTimeout=10 \
    -o StrictHostKeyChecking=accept-new \
    root@187.77.42.134 \
    'bash /root/scripts/h3mk-dispatch.sh' < /tmp/h3mk-dispatch.json)
  DISPATCH_EXIT=$?
  ```

  Parse the result:
  ```bash
  node -e "
  const r = JSON.parse(process.argv[1]);
  if (r.status === 'ok') {
    console.log('SUCCESS postId=' + r.postId);
  } else {
    console.error('FAILED code=' + r.code + ' msg=' + r.message);
    process.exit(1);
  }
  " -- "\$DISPATCH_RESULT"
  ```

  On SUCCESS: set `status: "done"`, log `postId` in `artifacts`. On FAILURE: apply Step 3f dead-letter logic.

- **`type: 'sasha-post-telegram'`** [h3mk runtime only] — Post to Sasha's Telegram channel. This action type is NOT YET WIRED in `h3mk-dispatch.sh`. Re-queue the item without incrementing attempts or dead-lettering. Log:
  ```
  [watch-queue] Skipping sasha-post-telegram ${item.taskId}: Telegram posting not yet wired — re-queuing
  ```
  Write the item back to `queues.h3mk` (append to end):
  ```
  node -e "
  const fs=require('fs');
  const f='social/dispatch-queue.json';
  const raw=fs.readFileSync(f,'utf8');
  let d;try{d=JSON.parse(raw)}catch(e){d={queue:[]}}
  const item=JSON.parse(process.argv[1]);
  if(d.schema==='v2'){
    if(!d.queues['h3mk'])d.queues['h3mk']=[];
    d.queues['h3mk'].push(item);
  } else {
    if(!d.queue)d.queue=[];
    d.queue.push(item);
  }
  fs.writeFileSync(f,JSON.stringify(d,null,2));
  " -- '<item JSON>'
  ```
  Do NOT increment `item.attempts`. Do NOT dead-letter.

- **`type: 'wp-publish'`** [WordPress sites only — not mangabeira.net] — Publish content to a WordPress site using the `wordpress` skill. For projects that run on WordPress (e.g., EkkoGreen, Salto Financeiro when they migrate to WP) rather than the mangabeira.net Supabase/Lovable stack.

  Read the following fields from the claimed item:
  - `item.wpSiteUrl` — the WordPress site base URL (required, e.g. `https://ekkogreen.com.br`)
  - `item.wpUsername` — WordPress username for Application Password auth (required)
  - `item.postTitle` — post title string
  - `item.postBodyFile` — path to a file containing the post body HTML or markdown (use the `--file` pattern; do NOT interpolate content into shell arguments)
  - `item.postStatus` — `"draft"` or `"publish"` (default `"draft"` if absent — never auto-publish without an explicit `"publish"` value in the queue item)

  Validate required fields before running the script. If `item.wpSiteUrl` or `item.wpUsername` is missing, skip and dead-letter immediately (treat as `attempts >= 3` — these are configuration errors, not transient failures).

  Write `item.postTitle` to a temp file using Node (safe pattern — never shell-interpolate untrusted content):
  ```bash
  node -e "require('fs').writeFileSync('/tmp/dispatch-wp-title.txt', process.argv[1])" -- "<item.postTitle_raw_value>"
  ```

  Then run:
  ```bash
  node .claude/skills/wordpress/scripts/wp-post.js \
    --site-url "<item.wpSiteUrl>" \
    --username "<item.wpUsername>" \
    --title-file /tmp/dispatch-wp-title.txt \
    --file "<item.postBodyFile>" \
    --status "<item.postStatus or 'draft'>"
  ```

  // Script at .claude/skills/wordpress/scripts/wp-post.js — to be created per wordpress skill docs

  Note: `item.wpSiteUrl` and `item.wpUsername` are operator-configured values (not user-generated content), so they may be passed as direct arguments. `item.postBodyFile` is a file path — pass it as `--file` so the script reads content from disk; do not read the file content into a shell variable. `item.postTitle` is user content and must be written to a temp file first, as shown above.

  On SUCCESS (zero exit from the script): set `status: "done"` and log the returned WordPress post ID in `artifacts` (e.g., `{ "wpPostId": 123, "wpPostUrl": "https://ekkogreen.com.br/?p=123" }`).

  On FAILURE (non-zero exit from the script): do NOT set `status: "done"`. Proceed with Step 3f dead-letter logic (increment `item.attempts`, dead-letter if `>= 3`, re-queue with backoff if `< 3`).

- **`type: 'read-later'`** — Process a queued read-later URL using the `read-later-capture` skill.

  **Note: This case is LLM-executed — the watch-queue agent invokes the skill directly via Claude's Skill tool, not a bash subprocess.**

  Read the following fields from the claimed item:
  - `item.url` — the URL to process (required; dead-letter immediately if absent or malformed)
  - `item.category` — optional category hint: `"research"`, `"content-idea"`, `"competitor"`, `"protocol-data"`, `"news"` (default: `"research"`)
  - `item.notes` — optional human annotation from Gabriel at queue time
  - `item.taskId` — optional; if present, the skill will update the task in `social/tasks.json`

  Validate `item.url` before invoking the skill. If `item.url` is absent or not a valid HTTP/HTTPS URL, treat as a configuration error: dead-letter immediately (same as `attempts >= 3`). Log:
  ```
  [watch-queue] Dead-lettering read-later item ${item.id}: invalid or missing URL
  ```

  Invoke the skill by calling the Skill tool with name `read-later-capture` and passing:
  - `url`: the value of `item.url`
  - `category`: the value of `item.category` or `"research"` if absent
  - `notes`: the value of `item.notes` or `""` if absent
  - `taskId`: the value of `item.taskId` or `null` if absent

  The skill handles scraping (Firecrawl with Apify fallback), insight extraction, file routing, and task update in `social/tasks.json`.

  On SUCCESS (skill returns an output file path and one-line summary): set `item.status = "done"` and log the output file path in `artifacts` (e.g., `{ "research_file": "research/read-later/2026-05-15-example-article.md" }`).

  On FAILURE (skill returns an error or the URL is invalid): do NOT set `status: "done"`. Proceed with Step 3f dead-letter logic (increment `item.attempts`, dead-letter if `>= 3`, re-queue with backoff if `< 3`).

  If the skill enters stub mode (scrape failed, stub file written with `qaStatus: "hold"`): treat this as a partial success — set `item.status = "done"` so the item is not re-queued, but log `[watch-queue] read-later ${item.id}: scrape failed, stub written — qaStatus=hold requires manual review`.

Print: `[watch-queue] Dispatching: [taskId or id] — type: [type or stage or ops]`

### 3c — Execute

Run the matching command above with its arguments. Capture:
- `status` — "done" / "failed" / "partial"
- `summary` — one-line description of what happened
- `artifacts` — array of URLs or file paths produced
- `errors` — array of error strings if any

### 3d — Write result file (only if dispatch came from OpenCLAW)

If the claimed item has `source: "openclaw"`, write a result file at `social/dispatch-results/<id>.json` so the VPS-side drain.mjs cron can post the result back to the originating Slack thread.

Result file schema (matches `Maestro-OpenClaw/workspace/skills/read-marketing-results/scripts/drain.mjs`):

```json
{
  "id": "<same id as the dispatch>",
  "slackChannel": "<from dispatch>",
  "slackThreadTs": "<from dispatch, may be null>",
  "slackBotAccount": "default",
  "status": "done | failed | partial",
  "summary": "short one-liner shown in Slack header",
  "artifacts": ["https://...", "/path/to/file.md"],
  "errors": ["error string"]
}
```

Then commit + push so the VPS drain cron picks it up on its next 60s tick:

```bash
mkdir -p social/dispatch-results
# (write the JSON file via Write tool, file path: social/dispatch-results/<id>.json)
git add social/dispatch-results/
git commit -m "result: <id> — <one-line summary>" 2>/dev/null && \
  git push origin main 2>/dev/null || echo "[watch-queue] Note: result push deferred — will retry next tick"
```

Skip this step entirely if `source` is anything other than `"openclaw"` (legacy local-only dispatches don't need a result file).

### 3e — Repeat

After execution and result writeback complete, check if any items remain in the queue. If yes, repeat from Step 3a.

If no items remain, remove the lock:
```
node -e "try{require('fs').unlinkSync('social/dispatch-queue.lock')}catch(e){}"
```

### 3g — Probabilistic Slack QA ping (post-dispatch hook)

After a dispatch action completes **successfully** (any action type, any `status: "done"` outcome), run the following sampling check:

1. Read `_ops/cost-config.json`. Extract `sample_review_pct` (default `0.15` if the field is absent or the file cannot be read).
2. Generate a random number between 0 and 1.
3. If the random number is less than `sample_review_pct`, send a Slack notification via the `chat.postMessage` API endpoint (`https://slack.com/api/chat.postMessage`) using `SLACK_BOT_TOKEN` and `SLACK_CHANNEL_ID` from the `.env` file.

The message must include: task id, task title, action type, and the text: `Random QA sample — please spot-check this output.`

Example curl call (write to a temp file, then curl — do not shell-interpolate untrusted content directly):

```bash
# Write task id, title, and action type to temp files using Node process.argv
# (safe pattern: values are passed as Node arguments, never shell-interpolated into quoted strings)
node -e "require('fs').writeFileSync('/tmp/dq-task-id.txt', process.argv[1])" -- "${item.id}"
node -e "require('fs').writeFileSync('/tmp/dq-task-title.txt', process.argv[1])" -- "${item.title||item.taskId||item.id}"
node -e "require('fs').writeFileSync('/tmp/dq-action.txt', process.argv[1])" -- "${item.type||item.stage||'ops'}"

node -e "
const fs = require('fs');
const cfg = (() => { try { return JSON.parse(fs.readFileSync('_ops/cost-config.json','utf8')); } catch(e) { return {}; } })();
const pct = typeof cfg.sample_review_pct === 'number' ? cfg.sample_review_pct : 0.15;
const token = process.env.SLACK_BOT_TOKEN;
const channel = process.env.SLACK_CHANNEL_ID;
if (!token || !channel) process.exit(0);
if (Math.random() >= pct) process.exit(0);
const taskId = fs.readFileSync('/tmp/dq-task-id.txt', 'utf8');
const title = fs.readFileSync('/tmp/dq-task-title.txt', 'utf8');
const action = fs.readFileSync('/tmp/dq-action.txt', 'utf8');
const msg = { channel, text: 'Random QA sample — please spot-check this output.\nTask: ' + taskId + '\nTitle: ' + title + '\nAction: ' + action };
fs.writeFileSync('/tmp/slack-qa-payload.json', JSON.stringify(msg));
process.exit(2); // signal: fire the ping
"
EXIT_CODE=$?
if [ $EXIT_CODE -eq 2 ]; then
  curl -s -X POST https://slack.com/api/chat.postMessage \
    -H "Authorization: Bearer ${SLACK_BOT_TOKEN}" \
    -H "Content-Type: application/json" \
    --data @/tmp/slack-qa-payload.json >/dev/null 2>&1 || true
fi
```

**Rules:**
- If `SLACK_BOT_TOKEN` or `SLACK_CHANNEL_ID` is not set, skip silently (no error, no log).
- If the file read or curl call fails for any reason, skip silently — this hook must never block or fail the dispatch loop.
- This hook only fires on SUCCESS. Failed dispatches skip this step entirely (they go to 3f instead).

### 3f — Dead-letter on repeated failure

If an item's execution returns `status: "failed"`, increment `item.attempts` (default 0). If `item.attempts >= 3`, remove it from the queue and write it to `deadLetter` with `failReason` and timestamp:

```
node -e "
const fs=require('fs');
const f='social/dispatch-queue.json';
const item=JSON.parse(process.argv[1]);
const d=JSON.parse(fs.readFileSync(f,'utf8'));
item.attempts=(item.attempts||0)+1;
item.lastFailedAt=new Date().toISOString();
if(item.attempts>=3){
  item.deadLetteredAt=new Date().toISOString();
  item.failReason=item.failReason||'execution-failed';
  if(!d.deadLetter)d.deadLetter=[];
  d.deadLetter.push(item);
  console.log('DEAD_LETTERED');
} else {
  // Re-queue with incremented attempts
  if(d.schema==='v2'){
    const rt=item.runtime||'mrzq';
    if(!d.queues[rt])d.queues[rt]=[];
    d.queues[rt].push(item);
  } else {
    if(!d.queue)d.queue=[];
    d.queue.push(item);
  }
  console.log('REQUEUED');
}
fs.writeFileSync(f,JSON.stringify(d,null,2));
" -- '{...item JSON...}'
```