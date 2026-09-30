---
name: commit-push
description: Stage, commit, and push in one step. Auto-generates the commit message and pushes immediately by default; pass any flag (e.g. -c) to review/edit the message first.
triggers:
  - commit and push
  - commit push
  - commit then push
  - save and push
allowed-tools:
  - Bash
  - AskUserQuestion
---

## What this skill does

Stages all changes, commits, then immediately pushes to the remote.
By default it auto-generates the commit message and just does it — no
prompts. Pass any flag as an argument (e.g. `-c`, `-a`, `-confirm` —
the value doesn't matter, only that it starts with `-`) to instead be
shown the generated message with a chance to approve, edit, or write
your own.

---

## Steps

### 1. Determine mode

- **No argument** → **auto mode** (default): generate the message and push without asking.
- **Any argument starting with `-`** → **confirm mode**: generate the message, then let the user approve, edit, or replace it before committing.

### 2. Check there is something to commit

```bash
git status
git diff --stat HEAD
```

If the working tree is clean with nothing staged or untracked, say so and stop.

### 3. Generate the commit message

Run:
```bash
git diff HEAD
git diff --cached
```

Read the diff carefully. Write a concise conventional commit message:
- Format: `type: short summary` (under 72 chars)
- Types: feat, fix, refactor, chore, docs, style
- Optional body if the change is non-obvious (blank line after subject, then body)

### 4. Auto mode — just do it

Stage, commit, and push without stopping to ask:

```bash
git add -A
git commit -m "<generated message>"
git push
```

Skip straight to step 6 to report the result.

### 4′. Confirm mode — review first

Present the generated message to the user exactly as it would be committed.
Use AskUserQuestion with three options:

- **A) Looks good — commit it**
- **B) Let me edit it** — ask for their revised version, then commit
- **C) Cancel** — stop without committing or pushing

Once approved (A or B):
```bash
git add -A
git commit -m "<message>"
```

Then continue to step 5 (push).

### 5. Push

```bash
git push
```

If that fails because no upstream is set, run:
```bash
git push -u origin $(git branch --show-current)
```

If it fails for any other reason (rejected, auth error, etc.), report the error
and stop — do not retry or force-push.

### 6. Report result

Show the commit hash, subject line, and confirm the push succeeded. Example:
`Committed and pushed: a3f9c12 — refactor: extract shared UI primitives → origin/main`
