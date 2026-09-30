---
name: polish-notes
description: Rewrite a raw/shorthand lecture note into the polished format used across the vault (proper math notation, topic sections, tables, callouts). Defaults to the most recently modified lecture note.
triggers:
  - polish my notes
  - clean up my notes
  - clean up my lecture notes
  - format my lecture notes
  - polish this lecture note
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
---

## What this skill does

Takes a raw, shorthand lecture note (scribbled during class — abbreviated math,
fragment sentences, no structure) and rewrites its `## Notes` section into the
polished shape already used for notes like
`2 Areas/School/DATA 140/Lecture Notes/2026-09-03.md`. It does not invent
content: it reorganizes, expands notation, and clarifies what's already there.

## Steps

### 1. Identify the target note

If the user names a course or file, use that. Otherwise find the most recently
*modified* file under any course's `Lecture Notes/` folder:

```bash
find "$HOME/obsidian/2 Areas/School" -path "*/Lecture Notes/*.md" -print0 \
  | xargs -0 ls -t | head -1
```

Read that file.

### 2. Read one polished example for style calibration

Read `~/obsidian/2 Areas/School/DATA 140/Lecture Notes/2026-09-03.md` as the
reference style (if the target note *is* that file, pick another already-polished
note instead). Match its conventions:

- `### Subheadings` per topic, in the order topics were covered
- Inline math as `$...$`, display math as `$$...$$` — expand shorthand like
  `sigma`, `->`, `p^k(1-p)^(n-k)` into real LaTeX
- Tables for distributions, comparisons, or anything tabular in the raw notes
- A `> [!tip]` or `> [!note]` callout where the raw notes contain a "gotcha,"
  a common misconception, or a point the professor emphasized — not on every
  section, only where it earns its place
- Worked examples kept as their own labeled subsection
- Bold key terms on first use (e.g. **independent**, **Boole's inequality**)

### 3. Rewrite

- Keep frontmatter as-is (`type`, `course`, `date`, `lecture`) except: update
  `tags` to reflect the actual topics covered (lowercase, kebab-case, e.g.
  `[lecture, probability, binomial]`), matching the style of existing notes.
- Fill in `## Topics` with a bullet per topic actually covered, if it's empty
  or thin.
- Rewrite `## Notes` completely: organize into `### ` subsections by topic,
  clean up the math, keep every substantive point from the raw version — don't
  drop content, just restructure and clarify it. Fix obvious shorthand/typos
  (e.g. `histrogram` → `histogram`, `/sigma` → `\sum`) but don't add facts that
  aren't implied by what's there.
- If the raw notes cut off mid-thought (e.g. trail off after "so spread is"),
  leave a `> [!note]` flagging it's incomplete rather than inventing an ending.
- Leave `## Questions / Follow-ups` alone unless the raw notes clearly implied
  an open question worth capturing.
- If the lecture covered a clear, nameable topic, update the note's `# Title`
  and the `lecture:` link text on the course hub note to match, e.g.
  `# STAT 134 — Lecture 5: Binomial Mean and Variance` — matching how DATA 140
  titles its lectures.

### 4. Preserve the raw version

Before overwriting anything, save the original file's full content (frontmatter
and all, exactly as read in step 1) to a sibling file in the same folder named
`<original filename> (raw).md` — e.g. `2026-09-10.md` → `2026-09-10 (raw).md`.
This is a plain capture, not a new note type: keep its frontmatter as-is (don't
reclassify `type`), so it doesn't show up as a separate entry in any `.base`
dashboard.

If a `(raw)` sibling already exists for this note (re-polishing an
already-polished note), don't overwrite it — that would destroy an earlier
capture. Leave it in place and skip this step.

### 5. Save and report

Write the rewritten note to the original file path (overwrite), with one
addition: at the very bottom, after all existing sections, add:

```
---
*Raw notes: [[<original filename> (raw)]]*
```

Update the matching link text on the course hub note (e.g.
`STAT 134/STAT 134.md`) if the title changed. Report the file path, the path
to the preserved raw note, and a one-line summary of what topics it now
covers — don't paste the whole rewritten note back into the conversation.
