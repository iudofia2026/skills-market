---
name: work-logs
description: Write a daily work log for the current repo using a date-based archive hierarchy. Always lives at `docs/work logs/`. Creates the root and `README.md` if missing. Use when user says "/work-logs", "log today's work", "write a work log", "add a daily log", or wants to document what was done today in the current repo.
allowed-tools: ["Bash", "Read", "Write"]
---

Append today's work log to the current repository's canonical work-log archive at `docs/work logs/`. Creates the directory tree and seeds a `README.md` documenting the format on first use, then writes a single entry per calendar day at `docs/work logs/YYYY/MM-month/DD/MON_DD.md`.

## Execute

### Step 1: Detect repo + today's date

```bash
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null) || {
    echo "✗ Not in a git repo. /work-logs only runs inside a git repository."
    exit 1
}

YEAR=$(date '+%Y')
MONTH_NUM=$(date '+%m')
MONTH_NAME=$(date '+%B' | tr '[:upper:]' '[:lower:]')
DAY=$(date '+%d')
MONTH_ABBR=$(date '+%b' | tr '[:lower:]' '[:upper:]')

LOG_ROOT="$REPO_ROOT/docs/work logs"
LOG_DIR="$LOG_ROOT/$YEAR/$MONTH_NUM-$MONTH_NAME/$DAY"
LOG_FILE="$LOG_DIR/${MONTH_ABBR}_${DAY}.md"
README="$LOG_ROOT/README.md"

echo "→ work-logs"
echo "  • repo: $(basename "$REPO_ROOT")"
echo "  • path: $LOG_FILE"
```

### Step 2: Ensure root + README exist

```bash
if [ ! -d "$LOG_ROOT" ]; then
    mkdir -p "$LOG_ROOT"
    echo "  ✓ created docs/work logs/"
fi

if [ ! -f "$README" ]; then
    # Seed README with format documentation (see "README template" below)
    # Use the Write tool to author it.
    echo "  ✓ seeded README.md"
fi

mkdir -p "$LOG_DIR"
```

### Step 3: Write or append today's log

If `$LOG_FILE` already exists, the agent should **append a new section** under a `## Update — HH:MM` header rather than overwriting. If it does not exist, author the full log following the canonical template (see below).

```bash
if [ -f "$LOG_FILE" ]; then
    echo "  ~ log exists — append a new ## Update section"
else
    echo "  ✓ creating new log"
fi
```

**CRITICAL:** Always Read the existing file first before writing. If `$LOG_FILE` exists, you MUST append content to it using Write (reading first, then reconstructing full content with new section appended). NEVER overwrite an existing daily log.

### Step 4: Commit work-log changes

**CRITICAL: Only commit work-log files. Never stage other changes.**

After writing/updating the log file:

```bash
cd "$REPO_ROOT"

# ─────────────────────────────────────────────────────────────────────
# GUARDRAIL: Verify ONLY work-log files are staged
# ─────────────────────────────────────────────────────────────────────

# Reset any stray staged files first (safety measure)
git reset HEAD 2>/dev/null

# Stage ONLY the work-log directory — use exact path, no wildcards
git add "docs/work logs/"

# Verify: check what's actually staged
STAGED_FILES=$(git diff --cached --name-only)
if echo "$STAGED_FILES" | grep -qv "^docs/work logs/"; then
    # Something outside work logs is staged — ABORT
    echo "  ✗ ABORT: Non-work-log files would be committed:"
    echo "$STAGED_FILES" | grep -v "^docs/work logs/"
    git reset HEAD
    exit 1
fi

# ─────────────────────────────────────────────────────────────────────

# Check if there are changes to commit
if ! git diff --cached --quiet; then
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M')

    git commit -m "docs(work-logs): $TIMESTAMP"
    echo "  ✓ committed work-log changes"
else
    echo "  • no changes to commit"
fi
```

**Guardrail explanation:**
1. `git reset HEAD` — clears any accidentally staged files first
2. `git add "docs/work logs/"` — stages ONLY work-log directory (exact path, no wildcards)
3. Verification step — checks if anything outside `docs/work logs/` is staged
4. If non-work-log files detected → ABORT and reset, don't commit

**Why separate commits?**
- Work logs are documentation, not code changes
- Keeps commit history clean — docs changes separate from feature work
- Easy to cherry-pick or revert log entries without touching code

### Step 5: Complete

```bash
echo "→ complete"
echo "  ✓ $LOG_FILE"
```

---

## Output Format

```
→ work-logs
  • repo: my-app
  • path: /path/to/my-app/docs/work logs/2026/04-april/23/APR_23.md
  ✓ created docs/work logs/
  ✓ seeded README.md
  ✓ creating new log
→ complete
  ✓ /path/to/my-app/docs/work logs/2026/04-april/23/APR_23.md
```

---

## Path format

```
docs/work logs/YYYY/MM-month/DD/MON_DD.md
```

Example: `docs/work logs/2026/04-april/23/APR_23.md`

| Segment | Format | Example |
|---------|--------|---------|
| Year | `YYYY` | `2026` |
| Month dir | `MM-month` | `04-april` |
| Day dir | `DD` | `23` |
| Filename | `MON_DD.md` | `APR_23.md` |

- Month number always zero-padded (`04`, not `4`)
- Month dir name always full + lowercase (`april`)
- Day always zero-padded (`07`, not `7`)
- Filename uses uppercase 3-letter month + zero-padded day (`APR_07.md`)
- Hyphens only — no spaces, no camelCase, no underscores in dir names

---

## Log file template

Every log file follows this exact structure. The agent fills in workstreams based on the day's actual work — anything from the conversation, recent commits (`git log --oneline --since=midnight`), or user-provided notes.

```markdown
# MON_DD.md

**Date:** <Weekday, Month DD, YYYY>
**Repo:** <repo name> (+ any secondary repos touched)

---

## Summary

<1–3 sentence overview of the day's work.>

---

## 1. <Workstream title>

<Bulleted or prose description of what was done, key decisions, links
to files touched, and any follow-ups.>

---

## 2. <Next workstream>

...

---

## Files touched (rough count)

- **New files:** ...
- **Renamed:** ...
- **Deleted:** ...
- **Modified:** ...

---

## Notes for tomorrow

- <Loose ends, open questions, follow-up tasks.>
```

### Update / append pattern

If a log file already exists for today (the agent ran `/work-logs` earlier), append rather than overwrite:

```markdown
---

## Update — 18:30

<New workstream(s) since the previous entry.>
```

Use 24-hour local time in the header.

---

## README template

Authored once per repo, on first run. Generic enough to be identical across all repos.

```markdown
# Work Logs

Daily work logs, archived under `YYYY/MM-month/DD/`.

## Path format

\`\`\`
docs/work logs/YYYY/MM-month/DD/MON_DD.md
\`\`\`

Example: `docs/work logs/2026/04-april/23/APR_23.md`

- `YYYY` — 4-digit year
- `MM-month` — zero-padded month number + full lowercase name (`04-april`)
- `DD` — zero-padded day (`23`)
- Filename — `MON_DD.md`, uppercase 3-letter month + zero-padded day (`APR_23.md`)

## File format

Every log file follows the same structure: metadata header, numbered
workstream sections separated by `---`, and two trailing sections —
**Files touched** and **Notes for tomorrow**.

See the most recent log file in this directory for a working example.

## Rules

- One file per day. Name it after the calendar day, not the agent's wall-clock.
- Append a `## Update — HH:MM` section if running `/work-logs` more than once on the same day.
- Always end with **Files touched** and **Notes for tomorrow**.
- Code snippets, tables, and route maps welcome.
```

---

## Information sources for content

When authoring the log, the agent should pull from (in priority order):

1. **The current conversation transcript** — primary source of what was done.
2. **Recent commits in the repo** — `git log --oneline --since=midnight --author="$(git config user.email)"` for things done outside the conversation.
3. **Uncommitted changes** — `git status` to flag work-in-progress for the "Notes for tomorrow" section.
4. **User-provided notes** — anything the user dictates as part of the `/work-logs` invocation.

Don't fabricate workstreams. If the day was light, write a short log — three sentences and a one-line "Notes for tomorrow" is fine.

---

## Output after every run

After committing, **always output two summaries in this order:**

1. **Lengthened summary** — full prose recap of every workstream logged. One paragraph or numbered list per major workstream. Enough detail that someone reading only the chat can reconstruct what happened without opening the log file.

2. **Simplified summary** — short numbered list, one line per item, plain language. No code formatting, no jargon. Should read like you're telling a friend what you did today.

Use this exact heading structure:

```
---

## What Got Done

<lengthened summary — paragraphs or numbered list>

---

<simplified — numbered list, 1 sentence each>
```

---

## Anti-patterns

- ❌ Putting work logs anywhere other than `docs/work logs/`
- ❌ Naming files `2026-04-23.md` or `april-23.md` (must be `APR_23.md`)
- ❌ Overwriting an existing same-day log instead of appending an `## Update` section
- ❌ Inventing workstreams that didn't happen
- ❌ Using a non-zero-padded day or month
- ❌ Putting the log root at the repo root (`work logs/`) instead of under `docs/`
- ❌ Bundling work-log changes with feature commits — **always commit work logs separately with `docs(work-logs):` prefix**
- ❌ Forgetting to commit — **work logs must be committed immediately after writing**
