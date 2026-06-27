---
name: commit-push
description: Stage, commit, and push in one step — custom or auto-generated message.
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

Stages all changes, commits with a message you provide or approve,
then immediately pushes to the remote. Combines /commit and /push into one flow.

---

## Steps

### 1. Check there is something to commit

```bash
git status
git diff --stat HEAD
```

If the working tree is clean with nothing staged or untracked, say so and stop.

### 2. Show a brief summary of what will be staged

List the changed files (modified, new, deleted). Keep it short.

### 3. Ask how to handle the commit message

Use AskUserQuestion with two options:

- **A) I'll write the message** — you type it
- **B) Auto-generate one for me to approve** — generate from the diff, show it, then confirm

### 4A. Custom message path

Ask the user for their commit message, then run:

```bash
git add -A
git commit -m "<their message>"
```

### 4B. Auto-generate path

Run:
```bash
git diff HEAD
git diff --cached
```

Read the diff carefully. Write a concise conventional commit message:
- Format: `type: short summary` (under 72 chars)
- Types: feat, fix, refactor, chore, docs, style
- Optional body if the change is non-obvious (blank line after subject, then body)

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
