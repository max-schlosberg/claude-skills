---
name: unix-check
description: Audit a codebase for Unix philosophy violations — file length, folder size, and duplicated logic.
triggers:
  - unix check
  - check unix philosophy
  - audit codebase structure
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
---

## What this skill does

Audits the current working directory for three categories of violations and reports
findings with file paths and line numbers. Optionally fixes them if asked.

---

## Rules

1. **No file over 300 lines** — a file that long is doing too many things.
2. **No folder with more than 5 files** — too many files in one place signals missing sub-structure.
3. **No logic duplicated 3+ times** — if the same pattern appears 3 or more times, it belongs in a shared function/component.

---

## Steps

### 1. Find oversized files
```bash
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.swift" \) \
  ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/dist/*" ! -path "*/.next/*" \
  | xargs wc -l 2>/dev/null \
  | awk '$1 > 300 && $2 != "total"' \
  | sort -rn
```
Report each file with its line count. Note how far over 300 it is.

### 2. Find overpopulated folders
```bash
find . -type d ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/dist/*" ! -path "*/.next/*" \
  | while read dir; do
      count=$(find "$dir" -maxdepth 1 -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.swift" \) | wc -l)
      if [ "$count" -gt 5 ]; then echo "$count $dir"; fi
    done \
  | sort -rn
```
Report each folder with its file count and list the files inside.

### 3. Detect duplicated logic
This requires reading the code. For each oversized file found in step 1, and for key
shared directories (components, utils, helpers, lib):
- Read the file and look for repeated patterns: identical function signatures, repeated
  JSX blocks (3+ similar elements), repeated API call patterns, repeated validation logic,
  copy-pasted type definitions.
- Flag any block of 5+ lines that appears to be duplicated within or across files.
- Use grep to spot obvious repetition:
```bash
# Example: find repeated import patterns or function call clusters
grep -rn "pattern" . --include="*.ts" --include="*.tsx" ! -path "*/node_modules/*"
```

### 4. Report
Output a structured report:

```
## Unix Philosophy Audit

### Oversized Files (> 300 lines)
- src/components/BigComponent.tsx — 487 lines (+187)
- src/utils/helpers.ts — 342 lines (+42)

### Overpopulated Folders (> 5 files)
- src/components/ — 12 files
- src/utils/ — 8 files

### Duplicated Logic (used 3+ times)
- `formatDate()` pattern appears in 4 files — extract to src/utils/date.ts
- Error boundary JSX block repeated in 3 components — extract to ErrorBoundary component

### Summary
X violations found. Ask me to fix any of them.
```

### 5. Fix (only if user asks)
- **Oversized file**: Read it, identify logical sections, propose a split into smaller
  files, confirm with user before writing.
- **Overpopulated folder**: Propose sub-folder groupings, confirm before moving files.
- **Duplicated logic**: Extract to a shared utility/component, update all call sites,
  confirm before writing.

Always confirm the proposed fix before making changes. Never restructure silently.
