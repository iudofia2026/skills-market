---
name: archive-style
description: Reference skill — defines Isiah's standard date-based archive path convention. Use when any skill or agent needs to know how to organize files by date in the archive style (e.g. screenshots, recordings, notes). Call this when user says "archive style", "use the archive structure", or "organize by date like the archive".
allowed-tools: ["Bash"]
---

Defines the canonical date-based directory structure used across all of Isiah's tooling. This is a reference skill — it does not perform any action. Import this convention into other skills or call it to remind an agent of the structure.

---

## The Archive Style

Files are organized under a root directory using three nested levels:

```
[root]/
└── YYYY/                     ← 4-digit year
    └── MM-month/             ← 2-digit month number + full lowercase month name
        └── DD/               ← 2-digit zero-padded day
            └── [files]
```

### Examples

```
~/screenshots/2026/03-march/27/problem-lab-full.png
~/Recordings/2026/03-march/27/absoluterest-hero-scroll.mov
~/github/isiah.md/archive/2026/03-march/claude-skills-creation.md
```

> Note: the isiah.md archive omits the `/DD/` level — files sit directly in `MM-month/`. All other tools (screenshots, recordings) include the `/DD/` level.

---

## Bash — Build the Path

```bash
YEAR=$(date '+%Y')
MONTH_NUM=$(date '+%m')
MONTH_NAME=$(date '+%B' | tr '[:upper:]' '[:lower:]')
DAY=$(date '+%d')

# With day (screenshots, recordings, etc.)
ARCHIVE_PATH="$ROOT/$YEAR/$MONTH_NUM-$MONTH_NAME/$DAY"

# Without day (isiah.md archive style)
ARCHIVE_PATH="$ROOT/$YEAR/$MONTH_NUM-$MONTH_NAME"
```

---

## Format Rules

| Segment | Format | Example |
|---------|--------|---------|
| Year | `YYYY` | `2026` |
| Month | `MM-month` | `03-march`, `11-november`, `04-april` |
| Day | `DD` | `01`, `27` |

- Month number is always zero-padded (`03` not `3`)
- Month name is always full and lowercase (`march` not `Mar`)
- Day is always zero-padded (`01` not `1`)
- No spaces, no camelCase, no underscores — hyphens only

---

## Usage Pattern

This is a **reference convention** — agents should create archive-style roots **in the current repo**, not in `isiah.md` or global locations (unless the tool specifically requires it).

When working in a repo (e.g., `~/github/octa`), create the archive root there:
```
octa/
└── ui-screenshots/       ← root name varies by content
    └── 2026/
        └── 04-april/
            └── 08/
                └── [files]
```

**Reference implementation**: See `~/github/isiah.md/archive/` for examples of the structure in action.

---

## Usage in Other Skills

When a skill needs to write to an archive-style path, it should define the path inline using the bash above — it does not need to call this skill at runtime. This skill exists so agents and users have a named, stable reference to point at when discussing the convention.
