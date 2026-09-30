---
name: shadow-sync
description: Export new Shadow meeting transcripts into the Obsidian vault, with Claude writing the summary.
triggers:
  - sync shadow
  - export shadow meetings
  - shadow to obsidian
allowed-tools:
  - Bash
  - Read
  - Write
---

## What this skill does

Runs the Shadow → Obsidian export script for any meeting Shadow has finished
**transcribing** and hasn't been exported yet. The script only handles the
transcript file + screenshots — Claude always writes the structured notes/summary
file itself (never waits on or reads Shadow's own `writeMeetingNotes` AI output).
This is deliberate: Shadow's free plan may never generate that AI summary, so
gating export on it would mean some meetings never get synced at all.

## Steps

1. Run the export script (no arguments) and capture output. For each meeting
   listed, it prints: `convIdx`, title, date, the transcript file path (already
   written), the notes file path (**not yet written** — this is Claude's job),
   screenshot count/folder, and the exact `--mark` command to finalize that
   meeting once its notes are done.
   ```bash
   python3 ~/scripts/shadow_to_obsidian.py
   ```
   If it prints "No new meetings to export," stop here and report that.

2. For each meeting printed:
   a. Read the transcript file at the printed `Transcript:` path.
   b. If `Screenshots:` is nonzero, the script has already done a brief
      perceptual-hash scan of the raw screenshots and copied the distinct
      ones (consecutive near-duplicate frames collapsed away) into a `key/`
      subfolder, reported on the `Key shots:` line — Read those first; only
      fall back to the full unfiltered screenshots folder if `key/` seems to
      have missed something (e.g. a gradual slide transition that never
      cleared the dedup threshold). The Read tool renders images directly —
      don't just embed the `![](file://...)` links from the transcript
      unread. These are the actual board/slide/screen content at that
      moment, which transcript audio alone can miss or mishear — e.g. exact
      formulas, diagrams, code, or table values written on a board or slide.
      Use them as primary source alongside the transcript: when a
      screenshot's on-screen content differs from what the transcript audio
      suggests (a formula, a spelling, a number), trust the screenshot. This
      matters most for lecture notes, where the whole point is capturing
      board content the student may not have been able to see or copy down
      live. If `Key shots:` says Pillow isn't installed, either run
      `pip3 install --user Pillow` once and rerun the export step, or fall
      back to reading every screenshot in the full folder directly.
   c. Embed a handful of the most content-relevant key screenshots directly
      into the notes file itself, next to the section they support — not
      just left linked from the transcript. Use the same
      `![](file:///absolute/path/to/key/<filename>)` convention already used
      in the transcript (URL-encode spaces as `%20`). Be selective: embed
      screenshots that show something the prose can't fully capture (a
      diagram, a dense table, on-screen code) or that a correction/callout
      is directly citing — not every key screenshot needs to land in the
      notes, only the ones that would materially help someone reviewing this
      without rewatching the recording.
   d. Write a structured notes file at the printed `Notes path:` — the shape
      depends on where that path lands:
      - **Under `1 Projects/Internship/.../Meetings/`** (the default route):
        follow the established meeting convention (see any existing file in
        `~/obsidian/1 Projects/Internship/Meetings/` for the exact shape, e.g.
        frontmatter with `type: meeting`, `date`, `attendees`, `project`, `tags:
        [meeting, raw-export]`; a `> [!warning] Raw-transcript export` callout
        noting Shadow hadn't generated its own AI notes as of export time; then
        sections: TL;DR (if there's a clear throughline relevant to the user's
        own project work), Meeting Overview, Key Discussion Points (grouped by
        topic/project if it's a multi-topic meeting), Decisions Made, Action
        Items (table: Task / Owner / Due), Open Questions, Notable Verbatim
        (a handful of short direct quotes), and — if relevant — a section tying
        discussion points back to any active engineering/project plan.
      - **Under `2 Areas/School/<course>/Lecture Notes/`** (online-course
        lecture recordings, routed via `COURSE_ROUTES` in the script — e.g.
        DATA 144): use the **Lecture Note** template shape instead
        (`~/obsidian/Templates/Lecture Note.md`) — frontmatter with `type:
        lecture-note`, `course: "[[<Course>]]"`, `date`, `tags: [lecture,
        raw-export]`; a `> [!warning] Raw-transcript export` callout same as
        above; then `## Topics` (bullet list of what was covered), `## Notes`
        (organized, corrected summary of the lecture content — same bar as
        notes written from a live in-class capture: fix obvious verbal slips,
        add the actual formulas/definitions if the professor referenced slides
        not visible in transcript audio alone), a `## Recording` section
        linking the transcript (`[[<transcript file>]]`), and `##
        Questions / Follow-ups`. Also add a link to this note under **Lecture
        Notes** on the course's hub note (e.g. `2 Areas/School/DATA
        144/DATA 144.md`), matching how DATA 140 lecture notes are linked.
   e. Identify speakers by name where the transcript makes it unambiguous
      (someone addressed directly by name, or a `spkrName` already set in the
      DB — the script fills this in automatically when available). Don't
      guess a name you're not confident about; leave it as `Speaker N`
      rather than misattribute a quote.
   f. Once the notes file is written, mark that meeting exported:
      ```bash
      python3 ~/scripts/shadow_to_obsidian.py --mark <convIdx>
      ```

3. Report which meetings were exported (title + date + notes file path), or
   say "No new meetings" if nothing was new. Offer to open or discuss any of
   the synthesized summaries.
