---
name: gcal-pull
description: Fetch Google Calendar events for a date and inject them into the Obsidian daily note's Schedule section.
triggers:
  - pull calendar
  - sync calendar to obsidian
  - gcal pull
  - what's on my calendar
  - add calendar events to daily note
allowed-tools:
  - mcp__claude_ai_Google_Calendar__list_events
  - Read
  - Edit
  - Write
  - Bash
---

## What this skill does

Fetches events from Google Calendar and writes them into the `## 📅 Schedule` section of
the matching Obsidian daily note. Creates the section if it doesn't exist.

## Configuration

- **Primary calendar:** `maxschlosberg@berkeley.edu`
- **Timezone:** `America/Los_Angeles`
- **Daily note folder:** `/Users/maxschlosberg/obsidian/Daily/`
- **Daily note filename format:** `YYYY-MM-DD DayName.md` (e.g. `2026-06-27 Saturday.md`)
- **Schedule section header:** `## 📅 Schedule`

## Steps

1. **Determine the target date:**
   - If the user passed an arg (e.g. `/gcal-pull 2026-06-28`), parse it as the target date.
   - Otherwise use today's date (`2026-06-27` per context, or derive from the current date).

2. **List events** using `mcp__claude_ai_Google_Calendar__list_events`:
   - `calendarId`: `maxschlosberg@berkeley.edu`
   - `startTime`: target date at `T00:00:00`
   - `endTime`: target date at `T23:59:59`
   - `timeZone`: `America/Los_Angeles`
   - `orderBy`: `startTime`

3. **Format the event list.** For each event, produce a line:
   - Timed event: `- HH:MM–HH:MM · Title` (24h or 12h, be consistent with what's already in the note)
   - All-day event: `- (all day) · Title`
   - If no events: write `- No events scheduled.`

4. **Find or create the daily note file:**
   - Compute the day name (e.g. Saturday) from the target date.
   - File path: `/Users/maxschlosberg/obsidian/Daily/YYYY-MM-DD DayName.md`
   - If the file doesn't exist or is empty, **create it** by resolving the template at
     `/Users/maxschlosberg/obsidian/Templates/Daily Note.md` with the target date:
     - Replace `<% tp.date.now("YYYY-MM-DD") %>` → target date string (e.g. `2026-06-28`)
     - Replace `<% tp.date.now("dddd, MMMM D, YYYY") %>` → full date (e.g. `Sunday, June 28, 2026`)
     - The Thursday conditional `<% if (tp.date.now("d") == "4") { %>...<% } %>` — include the
       lines inside only if the target date is a Thursday; otherwise remove the block entirely
       (including the `<% if ... %>` and `<% } %>` lines).
     - Write the resolved content to the file using Write.
     - Then proceed to inject calendar events as normal.

5. **Read the note** and locate the `## 📅 Schedule` section:
   - If the section exists: replace everything between the `## 📅 Schedule` header and the
     next `##` header (or end of file) with the new event list. Preserve a blank line after
     the header and before the next section.
   - If the section does NOT exist: insert it after the `## 🎯 Top 3 today` section (or after
     the frontmatter block if that section is also missing). Add a blank line before and after.

6. **Write the updated note** using Edit.

7. **Mark this session as done** by running:
   `touch ~/.claude/gcal-pulled-YYYY-MM-DD` (using the target date)
   This prevents the morning hook from re-prompting for the rest of the session.

8. **Report** how many events were added and for which date.
