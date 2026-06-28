---
name: gcal-push
description: Create a Google Calendar event from natural language. Optionally logs it in the daily note.
triggers:
  - schedule event
  - add to calendar
  - gcal push
  - create calendar event
  - put this on my calendar
allowed-tools:
  - mcp__claude_ai_Google_Calendar__create_event
  - mcp__claude_ai_Google_Calendar__list_calendars
  - Read
  - Edit
  - Bash
---

## What this skill does

Creates a Google Calendar event from a natural language description. Optionally appends a
line to the current day's `## 📅 Schedule` section in the Obsidian daily note.

## Configuration

- **Primary calendar:** `maxschlosberg@berkeley.edu`
- **Timezone:** `America/Los_Angeles`
- **Daily note folder:** `/Users/maxschlosberg/obsidian/Daily/`
- **Daily note filename format:** `YYYY-MM-DD DayName.md`

## Steps

1. **Parse the event from args.** The user will describe an event in natural language, e.g.:
   - `/gcal-push Team lunch tomorrow 1pm-2pm at Cafe Milano`
   - `/gcal-push Study session Monday 3-5pm`
   - `/gcal-push dentist appointment June 30 at 9am, 1 hour`
   
   Extract:
   - `summary` (title) — required
   - `startTime` — required (ISO 8601, `America/Los_Angeles` timezone)
   - `endTime` — required (default to 1 hour after start if not specified)
   - `location` — optional
   - `description` — optional

   If you can't determine start time, ask the user before proceeding.

2. **Create the event** using `mcp__claude_ai_Google_Calendar__create_event`:
   - `calendarId`: `maxschlosberg@berkeley.edu`
   - `timeZone`: `America/Los_Angeles`
   - Fill in the parsed fields.

3. **Confirm creation** by reporting the event title, date, and time back to the user.

4. **Optionally update the daily note** (only if the event is today):
   - Find `/Users/maxschlosberg/obsidian/Daily/YYYY-MM-DD DayName.md` for the event's date.
   - If `## 📅 Schedule` exists, insert the new event line in chronological order within
     that section: `- HH:MM–HH:MM · Title`
   - If the section doesn't exist, skip the note update and mention it.
   - Ask the user if they'd like the note updated if the intent is ambiguous.
