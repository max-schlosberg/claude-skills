---
name: inbox-triage
description: Read the Obsidian inbox, route each item to the right note or skill, then clear the inbox.
triggers:
  - inbox triage
  - process inbox
  - clear inbox
  - go through inbox
  - triage inbox
allowed-tools:
  - Read
  - Edit
  - Write
  - Bash
  - AskUserQuestion
  - mcp__claude_ai_Google_Calendar__create_event
  - mcp__claude_ai_Google_Calendar__list_calendars
  - mcp__claude_ai_Google_Calendar__list_events
---

## What this skill does

Reads every item in `00 Inbox/Inbox.md`, routes each one to the right place (daily note,
existing project note, new note, Google Calendar, etc.), then clears the inbox back to its
blank template state.

## Configuration

- **Inbox path:** `/Users/maxschlosberg/obsidian/00 Inbox/Inbox.md`
- **Vault root:** `/Users/maxschlosberg/obsidian/`
- **Daily notes folder:** `/Users/maxschlosberg/obsidian/Daily/`
- **Daily note filename format:** `YYYY-MM-DD DayName.md`
- **Timezone:** `America/Los_Angeles`
- **Primary calendar:** `maxschlosberg@berkeley.edu`

---

## Steps

### 1. Read the inbox

Read `/Users/maxschlosberg/obsidian/00 Inbox/Inbox.md`.

Extract every bullet point or paragraph below the `# 📥 Inbox` heading. Ignore the "Dump
anything here" subtitle line. If the inbox is empty (no items), report that and stop.

---

### 2. Classify each item

For each inbox item, determine its type:

| Type | Examples | Action |
|---|---|---|
| **Calendar event** | "going to X at 5pm", "dentist Tuesday 10am", "meeting Friday 2-3pm" | Create GCal event + add to daily note schedule |
| **Task / intention** | "I want to clean my room tomorrow", "need to email prof" | Add to the relevant day's daily note (`## 🎯 Top 3 today` or `## ↪️ Tomorrow`) |
| **New note** | "idea for project X", "resource link", "job app for company Y" | Create the appropriate note in the right vault folder |
| **Existing note update** | "add X to project 1", "log Y in internship" | Find and edit the right note |
| **Ambiguous** | Unclear intent | Ask the user before acting |

A single inbox item may contain multiple sub-items — handle each sub-item independently.

---

### 3. Resolve relative dates

Today's date is provided in the session context as `currentDate`. Use it to resolve relative
references:
- "tomorrow" → today + 1 day
- "Monday" / day-of-week → next occurrence of that weekday
- "tonight" → today

Always compute the absolute date before acting.

---

### 4. Handle calendar events

For each calendar event:

1. Parse: `summary`, `startTime` (ISO 8601), `endTime` (default +1 hour if not given),
   `location` (optional), `description` (optional).
2. Call `mcp__claude_ai_Google_Calendar__create_event`:
   - `calendarId`: `maxschlosberg@berkeley.edu`
   - `timeZone`: `America/Los_Angeles`
3. Add a line to the relevant daily note's `## 📅 Schedule` section:
   - Format: `- HH:MM–HH:MM · Summary [· location if provided]`
   - Insert in chronological order within the section.
   - If the daily note doesn't exist yet, create it from the Daily Note template
     (copy the template from `/Users/maxschlosberg/obsidian/Templates/Daily Note.md`,
     replacing Templater tags with real values).

---

### 5. Handle tasks / intentions

For tasks intended for **today** → add to today's daily note `## 🎯 Top 3 today` or
`## 📝 Log` (whichever fits).

For tasks intended for **tomorrow or a future date** → add to today's daily note under
`## ↪️ Tomorrow`, or to that future day's daily note if it already exists.

Format: `- [ ] Task description`

---

### 6. Handle new notes

Create the note in the correct folder following vault conventions:

- Job application → `1 Projects/Job Applications/Companies/<Company>.md` using the
  Job Application template fields.
- Coding problem → `2 Areas/Coding Practice/Problems/<Name>.md` using the Coding Problem
  template fields.
- Project idea → `1 Projects/Side Projects/<Name>.md` using the Project template fields.
- Resource / reference → `3 Resources/<Name>.md` with `type: note`.
- Anything unclear → `00 Inbox/<Name>.md` and note it to the user.

Always include the minimal required frontmatter (`type`, `status`, `date`, etc.) matching
the note type from CLAUDE.md.

---

### 7. Handle existing note updates

Locate the target note with `find` or `ls`, read it, then edit the appropriate section.
Don't duplicate content already there.

---

### 8. Report what you did

Before clearing the inbox, output a brief summary in this format:

```
Inbox triage — YYYY-MM-DD

✅ Calendar events created:
  - [event title] on [date] at [time]

✅ Tasks added:
  - "[task]" → [daily note path]

✅ Notes created/updated:
  - [path]

⚠️  Skipped (ambiguous):
  - [item] — [reason]
```

---

### 9. Clear the inbox

Reset `00 Inbox/Inbox.md` to its blank state:

```markdown
---
type: note
tags: [inbox]
---

# 📥 Inbox

Dump anything here — links, ideas, half-thoughts. Sort into the right folder later.

```

---

### 10. Commit the vault

```bash
cd ~/obsidian && git add -A && git commit -m "inbox: triage $(date +%Y-%m-%d)"
```
