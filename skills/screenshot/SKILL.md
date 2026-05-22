---
name: screenshot
description: Capture a full-page screenshot + individual named-section screenshots of a running localhost page. Saves to ~/screenshots/YYYY/MM-month/DD/. Use when user says "/screenshot", "screenshot the page", "capture this page", or "screenshot [route]".
allowed-tools: ["Bash", "mcp__playwright__browser_navigate", "mcp__playwright__browser_take_screenshot", "mcp__playwright__browser_snapshot", "mcp__playwright__browser_evaluate"]
---

Capture a composite full-page screenshot and individual section screenshots of a localhost page. Organizes output under `~/screenshots/` with this exact structure:

```
~/screenshots/
└── YYYY/                        ← 4-digit year, e.g. 2026
    └── MM-month/                ← zero-padded month number + full lowercase month name, e.g. 03-march
        └── DD/                  ← zero-padded day, e.g. 27
            ├── [slug]-full.png
            ├── [slug]-[section].png
            └── ...
```

Examples of valid paths:
- `~/screenshots/2026/03-march/27/problem-lab-full.png`
- `~/screenshots/2026/03-march/27/problem-lab-confession.png`
- `~/screenshots/2026/04-april/01/homepage-full.png`

The bash to build this path:
```bash
YEAR=$(date '+%Y')
MONTH_NUM=$(date '+%m')
MONTH_NAME=$(date '+%B' | tr '[:upper:]' '[:lower:]')
DAY=$(date '+%d')
SCREENSHOTS_DIR="$HOME/screenshots/$YEAR/$MONTH_NUM-$MONTH_NAME/$DAY"
```

Do not reference any external file to determine this structure — it is fully defined above.

## Execute

### Step 1: Parse input and resolve URL

The user provides a URL or route. Examples:
- `/problem-lab` → `http://localhost:3000/problem-lab`
- `http://localhost:3333/problem-lab` → use as-is
- `problem-lab` → `http://localhost:3000/problem-lab`

If no port is specified, default to 3000. If the user specifies a port (e.g. `localhost:3333`), use that.

Confirm the resolved URL before proceeding.

### Step 2: Create dated output directory

```bash
YEAR=$(date '+%Y')
MONTH_NUM=$(date '+%m')
MONTH_NAME=$(date '+%B' | tr '[:upper:]' '[:lower:]')
DAY=$(date '+%d')

SCREENSHOTS_DIR="$HOME/screenshots/$YEAR/$MONTH_NUM-$MONTH_NAME/$DAY"
mkdir -p "$SCREENSHOTS_DIR"
echo "→ output: $SCREENSHOTS_DIR"
```

The path format is `YYYY/MM-month/DD` — e.g. `2026/03-march/27`. No external reference needed; the bash above generates it correctly.

### Step 3: Navigate to the page

Use `mcp__playwright__browser_navigate` with the resolved URL.

### Step 4: Capture full-page composite

Use `mcp__playwright__browser_take_screenshot` with:
- `fullPage: true`
- `filename`: `$SCREENSHOTS_DIR/[slug]-full.png` where slug is the route with `/` replaced by `-` (e.g. `problem-lab-full.png`)
- `type`: `"png"`

### Step 5: Discover named sections/anchors on the page

Use `mcp__playwright__browser_snapshot` to get the page structure, then scan for:
- Nav anchor links (`href="#something"`)
- Elements with `id` attributes that correspond to named sections

Also check if the page has an explicit nav bar listing section IDs.

Collect all section anchor IDs into a list.

### Step 6: Screenshot each section

For each section anchor found:
1. Navigate to `[base-url]#[anchor-id]` using `mcp__playwright__browser_navigate`
2. Wait briefly for scroll to settle
3. Capture viewport screenshot with `mcp__playwright__browser_take_screenshot`:
   - `filename`: `$SCREENSHOTS_DIR/[slug]-[anchor-id].png`
   - `type`: `"png"`
   - `fullPage: false` (viewport only — captures what's visible at that scroll position)

### Step 7: Report output

```bash
echo ""
echo "→ screenshots saved"
echo "  • $SCREENSHOTS_DIR"
ls "$SCREENSHOTS_DIR" | while read f; do
  echo "  ✓ $f"
done
echo ""
echo "→ complete"
```

---

## Output Format

```
→ output: ~/screenshots/2026/03-march/27/

→ capturing http://localhost:3333/problem-lab
  ✓ problem-lab-full.png        (full page composite)
  ✓ problem-lab-confession.png  (section A)
  ✓ problem-lab-chart.png       (section B)
  ✓ problem-lab-timeline.png    (section C)
  ✓ problem-lab-scroll.png      (section D)
  ✓ problem-lab-objects.png     (section E)
  ✓ problem-lab-environments.png (section F)

→ complete
```

---

## Archive Structure

```
~/screenshots/
└── 2026/
    └── 03-march/
        └── 27/
            ├── problem-lab-full.png
            ├── problem-lab-confession.png
            ├── problem-lab-chart.png
            └── ...
        └── 28/
            └── homepage-full.png
    └── 04-april/
        └── 01/
            └── elite-full.png
```

- `YYYY/` — 4-digit year
- `MM-month/` — zero-padded month number + full lowercase month name (e.g. `03-march`, `11-november`)
- `DD/` — zero-padded day (e.g. `01`, `27`)

Each day gets its own directory. Multiple captures from the same day accumulate in the same `DD/` folder, prefixed by their route slug so they don't collide.

---

## Notes

- If no anchor sections are found on the page, only the full-page composite is saved.
- If Playwright MCP is not available, report `✗ Playwright not available` and stop.
- Default port is 3000. Always prefer the port the user specifies.
- The slug is derived from the URL path: `/problem-lab` → `problem-lab`, `/elite` → `elite`, `/` → `home`.
- Full-page composite always saves first, then individual sections in nav order.
