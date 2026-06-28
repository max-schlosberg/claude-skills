---
name: obsidian-log
description: Write or update Obsidian internship logs — one per project, plus a daily log linking them.
triggers:
  - obsidian log
  - update obsidian
  - log to obsidian
  - write to obsidian
  - obsidian update
  - update my notes
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---

## What this skill does

Looks at git commits in the current project repo since the last log entry, writes a
project-specific log entry, and updates (or creates) the day's daily log with a link to it.

Vault lives at: `~/obsidian/`
Daily logs: `~/obsidian/1 Projects/Internship/Logs/YYYY-MM-DD Daily.md`
Project logs: `~/obsidian/1 Projects/Internship/Logs/<ProjectFolder>/YYYY-MM-DD.md`

Project folder mapping (update as new projects are added):
- Website repo → `Website/`
- Bioreactor repo → `Bioreactor/`

---

## Steps

### 1. Identify the current project

Determine which project the current working directory belongs to (e.g. the website repo maps
to `Website/`). If it's ambiguous, ask the user.

### 2. Find the last project log entry

```bash
ls ~/obsidian/1\ Projects/Internship/Logs/<ProjectFolder>/ | sort | tail -5
```

The most recent file's date gives the last-logged date. Extract it.

### 3. Get all git activity since that date

```bash
git log --since="<last-log-date>" --pretty=format:"%h %ad %s" --date=short
```

```bash
git log --since="<last-log-date>" --stat --pretty=format:"" | grep -v "^$" | head -40
```

If there are no new commits, say so and stop unless the user says to continue anyway.

### 4. Read project note for context

Read `~/obsidian/1 Projects/Internship/Project 1 - Website/Project 1 — Website.md`
(or the relevant project note) to understand open items and context.

### 5. Check if today's project log already exists

```bash
ls ~/obsidian/1\ Projects/Internship/Logs/<ProjectFolder>/ | grep $(date +%Y-%m-%d)
```

If it does, read it. Append only new bullets to relevant sections — do not duplicate
content already there.

### 6. Generate the log content

Write a rich project log using the commits, diffs, and project context.

Use this exact template:

```markdown
---
type: internship-log
date: YYYY-MM-DD
project: <ProjectName>
tags: [internship, log, <project-slug>]
---

# <ProjectName> Log — Weekday, Month DD, YYYY

## 🛠️ What I worked on
- ...

## 📦 Shipped / Progress
- ...

## 🚧 Blockers
- ...

## 🤝 People & meetings
- ...

## 📚 Learned today
- ...

## 🌟 Wins to remember
- ...

## ↪️ Next
- ...
```

Rules:
- `date:` is `YYYY-MM-DD`; `project:` is the display name (e.g. `Website`)
- Heading uses full weekday: "Friday, June 27, 2026"
- Emoji section headers — keep them
- If a section has nothing, write `—` not blank
- Be specific: name files, components, techniques
- One idea per bullet, no paragraph-length bullets

### 7. Show the draft to the user

Output the full note content as a markdown code block. Do this BEFORE calling
AskUserQuestion — the question must come after the draft is visible.

Then use AskUserQuestion:
- **A) Save it** — write as-is
- **B) Edit something** — ask what to change, update, then save
- **C) Cancel** — don't write anything

### 8. Save the project log

- New file: write to `~/obsidian/1 Projects/Internship/Logs/<ProjectFolder>/YYYY-MM-DD.md`
- Existing file: append new bullets to the right sections only

### 9. Update the daily log

Check if `~/obsidian/1 Projects/Internship/Logs/YYYY-MM-DD Daily.md` exists.

**If it exists:** read it and add/update the line for this project under `## Projects`.

**If it doesn't exist:** create it with this template:

```markdown
---
type: daily-log
date: YYYY-MM-DD
tags: [internship, daily-log]
---

# Daily Log — Weekday, Month DD, YYYY

## Projects

- [[<ProjectFolder>/YYYY-MM-DD|<emoji> <ProjectName>]] — <one-sentence summary>

## Notes
- —
```

The wiki link format is `[[<ProjectFolder>/YYYY-MM-DD|<emoji> <ProjectName>]]` — e.g.
`[[Website/2026-06-27|🌐 Website]]`.

Emoji suggestions by project:
- Website → 🌐
- Bioreactor → 🧪

### 10. Update the project note's Log section

Read the project note and append to `## 🪵 Log`:

```
- YYYY-MM-DD — <one-sentence summary of what shipped>
```

### 11. Commit the vault

```bash
cd ~/obsidian && git add -A && git commit -m "log: YYYY-MM-DD <ProjectName> — <one-sentence summary>"
```

### 12. Report

Confirm:
- The project log path written
- The daily log updated/created
- The project note line appended
- That the vault was committed
