---
name: rubber-duck
description: >
  Cross-model review via Codex CLI (GPT reviewing Claude's work).
  TRIGGER proactively when: you finish a multi-file plan, complete a large
  implementation (many edits across files), write a test suite, retry a
  failed fix 2+ times, or prepare destructive operations (migrations,
  deletions, prod config). Also on demand: "rubber duck", "second opinion",
  "critique this", "what am I missing", /rubber-duck.
  DO NOT TRIGGER when: single-file edit, typo fix, config value change,
  or trivial task.
---

# Rubber Duck Companion

You have a rubber duck on your desk: Codex CLI running a different model
family (GPT). Like the classic debugging practice — you explain what you
did, the duck listens, and sometimes it spots what you missed.

The duck reviews what Claude (you) just did and gives the user a simple,
honest take. Not a formal audit. A colleague's quick read.

## When to activate — automatic

Invoke the duck **without being asked** when:

- You just finished a plan touching 3+ files or non-trivial tradeoffs
- You wrote/edited 100+ lines of meaningful changes across multiple files
- You wrote a test suite and are about to declare done
- You've retried the same fix 2+ times
- You're about to do something destructive (migrations, deletions, prod config)

## When to activate — on demand

When the user says anything like "critique this", "second opinion",
"rubber duck", "what am I missing", "check this", or `/rubber-duck`.

## How to run the review

### Step 1: Show the duck
```
>o)
(_>
```

### Step 2: Build the review command silently

Write the full `codex exec` command to `/tmp/rubber-duck-run.sh` so the
user doesn't see the raw plumbing. The script should:

1. Determine context source (pick first that applies):
   - **Git repo with uncommitted changes:** pipe `git diff HEAD` + untracked files
   - **Git repo, clean:** pipe `git diff HEAD~1..HEAD` (if the repo has
     only one commit, fall back to `git show HEAD`)
   - **No git:** use the Read tool to read the relevant files from the
     conversation (recently read/edited files, or files the user mentioned),
     then write their contents into the script as a heredoc piped to codex

2. Pipe context (capped at `head -c 12000`) into `codex exec`:
   ```
   codex exec \
     --sandbox read-only \
     --ephemeral \
     --skip-git-repo-check \
     "PROMPT"
   ```

3. The prompt should be:
   ```
   You are a rubber duck reviewer — a colleague giving a quick honest take.
   Review the work provided via stdin. Be direct and plain-spoken.
   Focus on: things that look wrong, decisions that seem off, stuff that
   was missed, and anything that will be painful to fix later.
   Skip style nits. Skip praise for things that are fine.
   If everything looks solid, just say: looks good, ship it.
   EXTRA_INSTRUCTIONS
   ```
   Where EXTRA_INSTRUCTIONS is any user-provided arguments.

### Step 3: Run it cleanly

Execute with just:
```bash
bash /tmp/rubber-duck-run.sh
```
This keeps the Bash tool call clean — the user sees `bash /tmp/rubber-duck-run.sh`,
not a wall of shell code.

### Step 4: Handle errors

If codex exec fails (non-zero exit, auth error, empty output), tell the
user plainly: "Duck couldn't connect — [reason]." Don't fake a review.

If `codex` is not installed, tell the user:
"Duck needs Codex CLI. Install it with `npm install -g @openai/codex` and run `codex login`."

## How to present the result

**Do NOT use a formal report format.** No "Fixed / Dismissed / For your
decision" categories. The duck is a companion, not an auditor.

Read the Codex output, evaluate each point against your full context,
then write a short conversational summary. Like a colleague saying
"hey, I looked at this — here's what I noticed."

### Example tone:

```
>o)
(_>
```

Took a look at what you just did.

- The retry logic in `fetcher.ts:82` doesn't back off — it'll hammer
  the endpoint on transient failures. This project uses `p-retry`
  elsewhere, probably worth using it here too.

- `user.preferences` can be null but you're accessing `.theme` directly
  on line 112. Added optional chaining.

**Ship it** after switching to `p-retry`.

### Rules for the summary:

- **Breathe.** Always put a blank line between list items for readability.
- **Plain language.** No jargon headers, no bullet-point categories.
- **Only mention things that matter.** Skip anything the reviewer flagged
  that you know is wrong given your full context — just don't mention it.
- **If you fixed something**, say what and where in one line.
- **If something needs the user's call**, describe it plainly.
- **End with a bold bottom line.** One sentence. Actionable.
  Good: "**Ship it** after fixing the null check."
  Bad: "**Consider** whether this aligns with long-term architectural goals."

## What NOT to do

- Don't invoke for trivial changes (typo, comment, config value)
- Don't invoke more than once per checkpoint type (plan, implementation,
  tests) — but a complex task can legitimately hit all three
- Don't send the entire codebase — only the diff or relevant files
- Don't blindly apply all suggestions — you have more context than the duck
- Don't show the raw codex exec command to the user

## Prerequisites

- [Codex CLI](https://github.com/openai/codex) installed and authenticated
  (`npm install -g @openai/codex && codex login`)
