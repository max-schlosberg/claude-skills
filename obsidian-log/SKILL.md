---
name: obsidian-log
description: Write a new Obsidian internship log covering all work done since the last log entry.
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

Looks at git commits in the current project repo since the last internship log entry,
generates a new `internship-log` note covering everything done, and saves it to the vault.
Also appends a one-liner to the project note's Log section.

Vault lives at: `~/obsidian/`
Logs live at: `~/obsidian/1 Projects/Internship/Logs/`
Project notes live at: `~/obsidian/1 Projects/Internship/Project 1 - Website/`

---

## Steps

### 1. Find the last log entry

```bash
ls ~/obsidian/1\ Projects/Internship/Logs/ | sort | tail -5
```

The most recent file's name gives the last-logged date (format: `YYYY-MM-DD Internship Log.md`).
Extract the date from the filename.

### 2. Get all git activity since that date

From the current working directory (the project repo):

```bash
git log --since="<last-log-date>" --pretty=format:"%h %ad %s" --date=short
```

Also get a file-level summary of what changed:
```bash
git diff <first-commit-after-last-log>..HEAD --stat 2>/dev/null || git log --since="<last-log-date>" --stat --pretty=format:"" | grep -v "^$" | head -40
```

If there are no commits since the last log, say "No new commits since <date>" and stop
unless the user says to continue anyway.

### 3. Read the current project note for context

Read `~/obsidian/1 Projects/Internship/Project 1 - Website/Project 1 — Website.md`
to understand the project context and see what the open items are.

### 4. Determine the note date

Today's date becomes the new log's date. Check that a log for today doesn't already exist:
```bash
ls ~/obsidian/1\ Projects/Internship/Logs/ | grep $(date +%Y-%m-%d)
```

If one already exists, read it. Then:
- Only pull git commits that aren't already reflected in the existing note (compare commit
  hashes / messages against what's already written)
- Merge the new activity into the existing note by appending bullets to the relevant sections
  rather than replacing the whole file — e.g. add new items under `🛠️ What I worked on`,
  new shipped items under `📦 Shipped / Progress`, etc.
- Do not duplicate content that's already there.

### 5. Generate the log content

Using the commits, diffs, and project context, write a rich internship log entry.
Fill every section — don't leave blanks. Infer from the commit messages and diff stats
what was worked on, what shipped, and what was learned.

The note must use this exact template (replace all placeholders):

```markdown
---
type: internship-log
date: YYYY-MM-DD
tags: [internship, log]
---

# Internship Log — Weekday, Month DD, YYYY

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
- `date:` field is `YYYY-MM-DD` (today's date)
- Heading uses the full weekday: "Friday, June 27, 2026"
- If a section has nothing to fill in (e.g. no blockers, no meetings), write `—` not blank
- Be specific — name files, components, and techniques, not just "worked on the website"
- Keep bullets tight: one idea per bullet, no paragraph-length bullets

### 6. Show the draft to the user

Output the full note content as a markdown code block in your response so the user
can read it completely. Do this BEFORE calling AskUserQuestion — the question must
come after the draft is visible, not before.

Then use AskUserQuestion with three options:
- **A) Save it** — write the file as-is
- **B) I want to edit something** — ask what to change, update, then save
- **C) Cancel** — don't write anything

### 7. Save the log note

- If the file is **new**: write to `~/obsidian/1 Projects/Internship/Logs/YYYY-MM-DD Internship Log.md`
- If the file **already existed**: edit it in place — append the new bullets to the appropriate
  sections. Do not rewrite sections that haven't changed.

### 8. Update the project note's Log section

Read `~/obsidian/1 Projects/Internship/Project 1 - Website/Project 1 — Website.md`.

Find the `## 🪵 Log` section and append a new line at the bottom of it:
```
- YYYY-MM-DD — <one-sentence summary of what shipped today>
```

Write the updated project note back.

### 9. Commit the vault

```bash
cd ~/obsidian && git add -A && git commit -m "log: <YYYY-MM-DD> internship log — <one-sentence summary>"
```

### 10. Report

Confirm:
- The log file path that was written
- The line added to the project note
- That the vault was committed
