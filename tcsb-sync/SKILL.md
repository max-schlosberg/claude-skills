---
name: tcsb-sync
description: Sync theCoderSchool Berkeley (Pike13) sessions from the subscribed TCSB feed into merged "Work" shift blocks on Google Calendar — adds, extends, trims, and removes shifts.
triggers:
  - tcsb sync
  - sync work shifts
  - update work schedule
  - populate work
allowed-tools:
  - mcp__claude_ai_Google_Calendar__list_calendars
  - mcp__claude_ai_Google_Calendar__list_events
  - mcp__claude_ai_Google_Calendar__create_event
  - mcp__claude_ai_Google_Calendar__update_event
  - mcp__claude_ai_Google_Calendar__delete_event
---

## What this skill does

Reads Max's TCSB sessions from the Pike13 calendar feed (subscribed in Google Calendar) and
makes the `Work` blocks on his primary calendar match: one event per shift, built by merging
his 1-hour sessions.

## Configuration

- **Source (read-only feed):** `as85ncq0thlmbu5j9sgtptv34l7nbvra@import.calendar.google.com`
  (the `webcal://tcs-berkeley.pike13.com/my_calendar.ics…` subscription). If that ID
  disappears, find it with `list_calendars` (summary contains `tcs-berkeley.pike13.com`).
- **Target:** `maxschlosberg@berkeley.edu`, timezone `America/Los_Angeles`.
- **Shift event:** title `Work`, `colorId: 7` (Peacock), no location/description.
- **Don't** use Pike13 login, the browser, or Gmail — the feed has every session
  (Pike13 emails only report changes, never standing weekly students).

## Steps

1. **Range.** Default: today through the end of next month (or whatever range Max names).

2. **Read sessions.** `list_events` on the source calendar for the range
   (`timeZone: America/Los_Angeles`, `orderBy: startTime`, `pageSize: 250`). Each event is one
   session with real start/end times (summary like `In Person 2:1 Code Coaching with …`).

3. **Build shifts per day.** Sort that day's sessions and merge any whose gap is **≤ 1 hour**
   (back-to-back 4pm + 5pm → 4–6; Saturday 10, 11, then 1pm → 10–2, matching how Max logs
   Saturdays). Shift = earliest start → latest end.

4. **Read existing shifts.** `list_events` on the target, `fullText: "Work"`, same range;
   keep only events titled exactly `Work`.

5. **Reconcile each day:**
   - Computed shift and existing Work block match → nothing.
   - Times differ → `update_event` the existing block.
   - Shift with no block → `create_event`.
   - Block with no sessions that day → `delete_event` (Pike13 dropped/canceled them).
   - Never touch past days.

6. **Conflicts.** Also list non-Work events on the target for the range. If a shift overlaps
   an all-day trip or other commitment (e.g. an out-of-town block), **don't create it** —
   flag it so Max can ask TCSB for a sub, and create it only if he says so.

7. **Report** a compact table: date · shift · action (added / changed / removed / unchanged /
   skipped-conflict), plus any conflicts.
