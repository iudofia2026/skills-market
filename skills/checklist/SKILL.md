---
name: checklist
description: Parse natural language prompts into executable tasks, create a task list, and execute sequentially.
category: productivity
priority: 7
---

# Checklist

Parse a natural language prompt into logical, executable tasks. Creates a durable checklist artifact for complex work, creates a compact Claude Code task list using TaskCreate/TaskUpdate, waits for approval, then executes tasks sequentially — always ending with production-lint and a work-log update.

## Usage

```
/checklist [natural language description of what you want done]
```

**Examples:**
```
/checklist fix the navbar mobile menu, add a search bar, and refactor the API calls
/checklist build a contact form with validation and send email notifications
/checklist update the hero section animations and make sure the scroll triggers work on mobile
```

---

## How It Works

### Step 1: Parse & Reason

Before creating tasks, reason through the prompt:

1. **Identify all distinct work items** in the prompt
2. **Determine logical execution order** (dependencies, build order)
3. **Break apart complex requests** into atomic, executable steps
4. **Ensure every part of the prompt** maps to at least one task

**Example breakdown:**
```
Prompt: "fix the navbar, add dark mode, refactor API calls"

Reasoning:
- "fix the navbar" → 1 task (single cohesive change)
- "add dark mode" → 2 tasks (1. create theme provider, 2. add toggle component)
- "refactor API calls" → 2 tasks (1. create unified hook, 2. migrate components)

Final task list: 5 tasks + production-lint + work-logs
```

### Step 2: Create Durable Checklist Artifact When Needed

Before creating Claude Code tasks, decide whether the plan needs a durable artifact.

**Create an artifact if any are true:**
- More than 5 tasks
- Multi-repo work
- Long implementation details
- Architectural decisions or tradeoffs
- User asks for a durable plan/checklist
- Context compaction would make the task risky to continue from memory

**Artifact root (in the current repo):**

```
docs/checklists/YYYY/MM-month/DD/<slug>.md
```

Use archive-style date folders inside the current repo's `docs/checklists/`:

```bash
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
YEAR=$(date '+%Y')
MONTH_NUM=$(date '+%m')
MONTH_NAME=$(date '+%B' | tr '[:upper:]' '[:lower:]')
DAY=$(date '+%d')
CHECKLIST_DIR="$REPO_ROOT/docs/checklists/$YEAR/$MONTH_NUM-$MONTH_NAME/$DAY"
mkdir -p "$CHECKLIST_DIR"
```

**Checklist file template:**

```markdown
# Checklist — <Title>

**Date:** <Weekday, Month D, YYYY>
**Repo:** `<repo-name>`
**Status:** Planned / In progress / Completed
**Canonical checklist path:** `docs/checklists/YYYY/MM-month/DD/<slug>.md`

---

## Intent

What this work is trying to accomplish and why.

---

## Execution Plan

1. **Task title**
   - Full implementation details.
   - Dependencies or sequencing notes.

---

## Claude Code Task Mapping

- Task 1: `Short task-card subject`
- Task N: `Run production-lint skill`

---

## Paths Touched

- `<path>` — why it may be touched.

---

## Verification

- [ ] Production lint or equivalent verification.
- [ ] Git status reviewed.
- [ ] Any DB/API smoke checks completed.

---

## Outcome

Fill in at completion or link to the relevant work log.
```

Also create/update `docs/checklists/README.md` in the current repo if missing so the convention is documented.

### Step 3: Create Task List

Use `TaskCreate` to build the task list. Each task should:

- Have a clear, actionable `subject` (imperative verb)
- Include `description` with enough context to execute for simple jobs
- Be atomic (one logical unit of work)
- Have proper dependencies via `blockedBy` if needed

For artifact-backed jobs, keep task cards compact and reference the checklist artifact + task number instead of embedding the full spec:

```
TaskCreate({
  subject: "Create weather schema",
  description: "Follow `docs/checklists/2026/05-may/15/weather-context-db-sync.md`, Task 1.",
  activeForm: "Creating weather schema"
})
```

Use `activeForm` as a short present-tense label for the active task (2-6 words).

### Always append the canonical tail tasks (in this order)

Every `/checklist` execution ends with the same two tail tasks: production-lint, then a work-log update.

**Tail order:**

```
N-1: Run production-lint skill          [ALWAYS]
N:   Update /work-logs for the session  [ALWAYS]
```

**1. production-lint — always:**
```
TaskCreate({
  subject: "Run production-lint skill",
  description: "Execute /production-lint to verify TypeScript, ESLint, build, and dependencies before deployment.",
  activeForm: "Running production-lint"
})
```

For non-code checklists (markdown-only, infra, data), production-lint can resolve as N/A — but the task must still be created and explicitly closed with an N/A note in chat so the rule remains visible.

**2. Work-log update — always (final task):**
```
TaskCreate({
  subject: "Update /work-logs for session",
  description: "For each git repo touched in this session, run /work-logs to write or update today's MON_DD.md (docs/work logs/YYYY/MM-month/DD/). If no session work log exists yet for today in a repo, log the full session — not just this checklist.",
  activeForm: "Updating session work logs"
})
```

Repos touched = all git repos where files were edited, committed, or whose state was meaningfully inspected during the session.

### Step 4: Present the plan first (required)

Before any approval UI, **output the full plan in the chat** as a clear, numbered list:

- Task number, short title, and one-line intent
- Checklist artifact path when one was created
- Explicit mention of the final **production-lint** and **work-logs** tasks
- Notes on **order** if dependencies matter

The user should be able to read the plan without opening a widget. Then proceed to **Step 5**.

### Step 5: Present for Approval

After the plan is visible, use **AskUserQuestion**:

```
AskUserQuestion({
  questions: [{
    question: "Task list ready. Proceed with execution?",
    header: "Tasks",
    options: [
      { label: "Execute all", description: "Run all tasks sequentially" },
      { label: "Modify first", description: "Let me adjust the task list" }
    ],
    multiSelect: false
  }]
})
```

### Step 6: Execute

If the user approves, execute each task:

1. `TaskUpdate({ taskId, status: "in_progress" })`
2. If the task references a checklist artifact and details are unclear, re-read the artifact before editing
3. Do the work
4. `TaskUpdate({ taskId, status: "completed" })`
5. Move to next task

**Execution rules:**
- Complete each task fully before moving on
- If a task fails, stop and report the issue
- Don't skip tasks
- The `production-lint` task invokes the `/production-lint` skill (mark N/A explicitly if not applicable)
- The final work-log task invokes `/work-logs` per repo touched; log the full session if no log exists yet for today
- Do not stuff long results into task descriptions; put outcomes in chat, work logs, or the checklist artifact

---

## Task Breakdown Guidelines

| Prompt Pattern | How to Split |
|----------------|--------------|
| "add X and Y" | 2 tasks (one for each) |
| "build X with Y" | 2+ tasks (build base, add feature) |
| "fix X" | 1 task (unless X has multiple parts) |
| "refactor X to use Y" | 2 tasks (create Y, migrate X) |
| "update X across the app" | 2+ tasks (update core, update usages) |

---

## Example Session

**User:**
```
/checklist add a loading skeleton to the cards and fix the pagination bug
```

**Agent reasoning:**
1. "add loading skeleton to cards" → Create skeleton component, integrate into cards
2. "fix pagination bug" → Single bug fix task
3. Always add production-lint and work-log update

**Agent prints the plan:**
```
Task 1: Create CardSkeleton component
Task 2: Integrate skeleton into card grid with loading state
Task 3: Fix pagination bug (investigate and resolve)
Task 4: Run production-lint skill
Task 5: Update /work-logs for session
```

**Agent asks for approval (after the plan is visible) → User approves → Agent executes all tasks in order**

---

## Notes

- Natural language parsing means the agent interprets intent
- Complex prompts may generate 5-10+ tasks
- Simple prompts may only generate 2-3 tasks
- The agent can split one prompt item into multiple tasks if needed for logical execution order
- Complex tasks should have durable checklist artifacts in `docs/checklists/`
- Task cards should stay compact; detailed instructions belong in the artifact
- **Tail tasks are mandatory:** every checklist ends with `production-lint` → `/work-logs` update
- production-lint runs on every checklist (mark N/A explicitly if not applicable)
- The work-log task is **always last** and covers every repo touched in the session, logging the full session if no log exists yet for today
