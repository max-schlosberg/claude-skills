---
name: sync-skills
description: Commit and push any changes to ~/.claude/skills/ to GitHub.
triggers:
  - sync skills
  - push skills
  - save skills
  - commit skills
  - update skills repo
allowed-tools:
  - Bash
---

## What this skill does

Commits and pushes any new or modified skill files in `~/.claude/skills/` to
`github.com/max-schlosberg/claude-skills`. Run this after creating or editing any skill.

---

## Steps

### 1. Check for changes

```bash
cd ~/.claude/skills && git status --short
```

If nothing to commit, say so and stop.

### 2. Stage and commit

```bash
cd ~/.claude/skills && git add -A && git commit -m "chore: update skills"
```

Use a more descriptive message if it's obvious what changed — e.g.
`"feat: add obsidian-log skill"` or `"fix: obsidian-log show draft before asking"`.
Infer from `git diff --cached --stat` what changed.

### 3. Push

```bash
cd ~/.claude/skills && git push
```

### 4. Report

Confirm what was committed and that the push succeeded.
