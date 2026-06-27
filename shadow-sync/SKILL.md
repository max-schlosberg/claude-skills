---
name: shadow-sync
description: Export new Shadow meeting notes into the Obsidian vault.
triggers:
  - sync shadow
  - export shadow meetings
  - shadow to obsidian
allowed-tools:
  - Bash
  - Read
---

## What this skill does

Runs the Shadow → Obsidian export script. Any meeting that Shadow has fully processed
(transcript + AI notes generated) and hasn't been exported yet will be saved as a
Markdown note in `~/obsidian/1 Projects/Internship/Meetings/`.

## Steps

1. Run the export script and capture output.
2. Report which meetings were exported (filename + date), or say "No new meetings" if nothing is new.
3. If a meeting was exported, offer to open or summarize it.

```bash
python3 ~/scripts/shadow_to_obsidian.py
```
