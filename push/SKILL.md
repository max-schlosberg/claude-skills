---
name: push
description: Push the current branch to its remote.
triggers:
  - push
  - push this
  - push to remote
  - push my changes
allowed-tools:
  - Bash
---

## What this skill does

Pushes the current branch to its upstream remote. If no upstream is set,
pushes with `-u origin <branch>` to set it.

---

## Steps

### 1. Check state

```bash
git status
git log @{u}..HEAD --oneline 2>/dev/null || git log --oneline -5
```

Show how many commits ahead of the remote the branch is.
If there is nothing to push, say so and stop.

### 2. Push

```bash
git push
```

If that fails because no upstream is set, run:
```bash
git push -u origin $(git branch --show-current)
```

### 3. Report result

Show the remote URL and branch, and confirm the push succeeded.
If it fails for any other reason (rejected, auth error, etc.), report the
exact error and stop — do not retry or force-push.
