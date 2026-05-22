---
name: ss
description: Find the newest screenshot on Desktop or Downloads from the last 5 minutes, analyze it, then delete it. Use when user says "/ss", "new ss", or "screenshot triage".
allowed-tools: ["Bash", "Read", "Delete"]
---

Triage **recent screenshots** on **Desktop** and **Downloads**: pick the **single newest** image modified within the **last 5 minutes**, **analyze** it (vision-capable read), then **delete** that file. If nothing qualifies, report **no new ss** and touch nothing.

## When **not** to use `/ss`

- **Anything you must keep** (admissions, financial aid, legal, HR, receipts): save into a durable notes repo or archive folder as markdown and/or HTML; **do not** run `/ss` on those files — the skill **deletes** the image after triage.
- **Images pasted into chat** (Cursor, Slack, etc.): the discover script **never** sees them; analyze from the attachment, then ask where to archive if you want a durable copy on disk.
- **`/ss` is for disposable UI triage** (debug screens, transient errors), not evidence.

## Execute

### Step 1 — Discover (Bash)

Run on the user's machine; print status lines to **stdout** (symbols: `→` `•` `✓` `✗` `!`).

```bash
echo "→ /ss discover (Desktop + Downloads, mtime ≤ 5m)"

if [[ "$(uname)" != "Darwin" ]]; then
  echo "  ! non-macOS: using \$HOME/Desktop and \$HOME/Downloads (Linux/XDG may differ)"
fi

_mt() {
  if [[ "$(uname)" == "Darwin" ]]; then
    stat -f '%m' "$1" 2>/dev/null || echo 0
  else
    stat -c '%Y' "$1" 2>/dev/null || echo 0
  fi
}

DESKTOP="${HOME}/Desktop"
DOWNLOADS="${HOME}/Downloads"
CANDIDATES=()

for root in "$DESKTOP" "$DOWNLOADS"; do
  if [[ ! -d "$root" ]]; then
    echo "  • skip (not a dir): $root"
    continue
  fi
  while IFS= read -r -d '' f; do
    CANDIDATES+=("$f")
  done < <(find "$root" -maxdepth 1 -type f \( \
    -iname '*.png' -o -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.webp' -o -iname '*.heic' \
  \) -mmin -5 -print0 2>/dev/null)
done

if [[ ${#CANDIDATES[@]} -eq 0 ]]; then
  echo "  • no new ss found (no matching images in last 5 minutes under Desktop/Downloads, maxdepth 1)"
  echo "→ complete"
  exit 0
fi

BEST="" BEST_MT=0
for f in "${CANDIDATES[@]}"; do
  mt=$(_mt "$f")
  if (( mt >= BEST_MT )); then
    BEST_MT=$mt
    BEST="$f"
  fi
done

if [[ -z "$BEST" || ! -f "$BEST" ]]; then
  echo "  ✗ internal: candidate list non-empty but no best file"
  echo "→ complete"
  exit 1
fi

echo "  ✓ latest: $BEST"
echo "SS_PATH=$BEST"
echo "→ next: Read image → summarize → Delete this path"
```

**Agent:** parse the line **`SS_PATH=...`** from the script output. That path is the **only** file eligible for this run.

### Step 2 — Read

Use **Read** on `SS_PATH`. Describe UI text, errors, hostnames, IPs, and anything actionable. If the format is not readable (rare), say so and still offer to delete only if the user confirms (default: **do not delete** if you cannot analyze at all).

### Step 3 — Report

Answer the user's implied question ("what is this / what do I do?") in **short, ordered steps**. Redact secrets if any appear in the image.

### Step 4 — Delete

Use **Delete** on **exactly** the `SS_PATH` file from Step 1. Do **not** delete other files. If Step 1 printed **no new ss**, **skip Delete**.

```bash
echo "  ✓ deleted (if Delete tool ran on SS_PATH)"
echo "→ complete"
```

---

## Output format (examples)

**Found:**

```
→ /ss discover (Desktop + Downloads, mtime ≤ 5m)
  ✓ latest: /Users/you/Downloads/Screenshot 2026-04-26 at 11.09.34.png
SS_PATH=/Users/you/Downloads/Screenshot 2026-04-26 at 11.09.34.png
→ next: Read image → summarize → Delete this path
```

Then your analysis, then:

```
  ✓ deleted screenshot
→ complete
```

**Not found:**

```
→ /ss discover (Desktop + Downloads, mtime ≤ 5m)
  • no new ss found (no matching images in last 5 minutes under Desktop/Downloads, maxdepth 1)
→ complete
```

---

## Rules

- **Time window:** only files with **`find … -mmin -5`** (strictly "modified within last five minutes" on the local clock).
- **Locations:** only **`$HOME/Desktop`** and **`$HOME/Downloads`**, **`maxdepth 1`** (no nested folders).
- **Extensions:** `png`, `jpg`, `jpeg`, `webp`, `heic` (case-insensitive).
- **Tie-break:** if two files share the same mtime, pick the lexicographically **last** path after the loop as implemented (acceptable); rare in practice.
- **One file per `/ss`:** only the **newest** qualifying file.
- **Privacy:** delete **after** analysis so screenshots do not linger unless analysis failed and you chose not to delete.

## Anti-patterns

- ❌ Deleting without **Read** first (unless unreadable and user declines analysis)
- ❌ Scanning whole `$HOME` or iCloud paths
- ❌ Ignoring the **5-minute** window
- ❌ Deleting anything not equal to **`SS_PATH`**
