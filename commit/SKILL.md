---
name: commit
description: Stage and commit changes — custom message, or auto-generate one for approval.
triggers:
  - commit
  - commit this
  - commit my changes
  - make a commit
allowed-tools:
  - Bash
  - AskUserQuestion
---

## What this skill does

Stages all modified/untracked files and commits them. Lets you write your own
message or have one generated from the diff for your approval.

---

## Steps

### 1. Check there is something to commit

```bash
git status
git diff --stat HEAD
```

If the working tree is clean and there is nothing staged or untracked, say so and stop.

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
- **B) Let me edit it** — ask them to provide their revised version, then commit
- **C) Cancel** — stop without committing

Once approved (A or B):
```bash
git add -A
git commit -m "<message>"
```

### 5. Report result

Show the commit hash and the subject line. Example:
`Committed: a3f9c12 — refactor: extract shared UI primitives`
