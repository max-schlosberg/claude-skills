---
name: vault-push
description: Commit and push all changes in the Obsidian vault to GitHub.
triggers:
  - push the vault
  - commit the vault
  - update the vault
  - sync the vault
allowed-tools:
  - Bash
---

## What this skill does

Stages all changes in `~/obsidian`, commits with either a user-provided message or a
sensible default, and pushes to `origin main`.

## Steps

1. Run `git -C ~/obsidian status` to check for changes. If nothing to commit, say so and stop.
2. Determine the commit message:
   - If the user provided one (e.g. `/vault-push add meeting notes`), use it.
   - Otherwise, run `git -C ~/obsidian diff --stat HEAD 2>/dev/null || git -C ~/obsidian status --short` to get a brief sense of what changed, then generate a message in the format: `vault: <brief description> (YYYY-MM-DD)` — e.g. `vault: add bioreactor meeting notes (2026-06-27)`.
3. Stage all changes: `git -C ~/obsidian add -A`
4. Commit with the message (use HEREDOC format).
5. Push: `git -C ~/obsidian push`
6. Report what was committed (file count, commit message) and confirm the push succeeded.
