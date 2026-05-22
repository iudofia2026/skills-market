---
name: inspect
description: Deep inspect current repository — analyze architecture, patterns, data flow, and understand how the app works end-to-end. Use when user says "/inspect" or asks for project overview.
allowed-tools: ["Bash", "Read", "Glob", "Grep"]
---

Deep inspection of the **current working directory's** codebase. The agent must read key files to understand architecture, patterns, data flow, and how the application functions from backend to frontend.

## Execute

### 1. Quick Status (Bash)

```bash
# Check if we're in a git repo
if ! git rev-parse --git-dir > /dev/null 2>&1; then
    echo "✗ Not a git repository"
    exit 1
fi

REPO_ROOT=$(git rev-parse --show-toplevel)
REPO_NAME=$(basename "$REPO_ROOT")
cd "$REPO_ROOT"

echo "→ inspecting $REPO_NAME..."
echo ""

# Git status
branch=$(git branch --show-current 2>/dev/null || echo "detached")
commit=$(git rev-parse --short HEAD 2>/dev/null || echo "unknown")
date=$(git log -1 --format="%cr" 2>/dev/null || echo "unknown")
remote=$(git remote get-url origin 2>/dev/null || echo "no remote")
remote_name=$(echo "$remote" | sed 's/.*github.com[/:]//' | sed 's/.git$//' 2>/dev/null || echo "unknown")

echo "GIT"
echo "  • $branch @ $commit ($date)"
echo "  → $remote_name"
echo ""

# Project type detection
TYPE="unknown"
[ -f "package.json" ] && TYPE="node"
[ -f "tsconfig.json" ] && TYPE="typescript"
[ -f "next.config.js" ] || [ -f "next.config.mjs" ] && TYPE="nextjs"
[ -f "remotion.config.ts" ] && TYPE="remotion"
[ -f "Cargo.toml" ] && TYPE="rust"
[ -f "go.mod" ] && TYPE="go"
[ -f "requirements.txt" ] || [ -f "pyproject.toml" ] && TYPE="python"

echo "TYPE"
echo "  • $TYPE"
echo ""

# Quick structure
echo "STRUCTURE"
ls -la | head -20
echo ""
```

### 2. Read Project Config (Read Tool)

**MUST read these files to understand project:**

Use `Read` tool on:

1. **package.json** — scripts, dependencies, project name
2. **tsconfig.json** or equivalent — build config
3. **CLAUDE.md** — if exists, contains project-specific instructions
4. **README.md** — project overview

After reading, output:

```
CONFIG
  • Framework: [detected framework]
  • Build: [build command from package.json scripts]
  • Dev: [dev command]
  • Key deps: [list 3-5 most important dependencies]
```

### 3. Analyze Architecture (Glob + Read)

**MUST analyze directory structure and key files:**

Use `Glob` to find:
- Entry points: `**/index.{ts,tsx,js,jsx}`, `**/main.{ts,tsx,js,jsx}`, `**/app.{ts,tsx,js,jsx}`
- API routes: `**/api/**/*.{ts,tsx,js,jsx}`, `**/routes/**/*.{ts,tsx,js,jsx}`
- Components: `**/components/**/*.{tsx,jsx}`
- Pages: `**/pages/**/*.{tsx,jsx}`, `app/**/page.tsx`
- Database/Models: `**/models/**/*`, `**/db/**/*`, `**/schema*`
- Services: `**/services/**/*`, `**/lib/**/*`, `**/utils/**/*`
- Config: `**/config/**/*`, `*.config.{js,ts,mjs}`

Then use `Read` to examine:
- Main entry point (e.g., `src/index.ts`, `app/layout.tsx`)
- One API route example
- One component example
- Database schema if present

After analysis, output:

```
ARCHITECTURE
  • Entry: [main entry file path]
  • Routes: [number of API routes] endpoints
  • Components: [number] components
  • Database: [type if found, e.g., "PostgreSQL via Prisma"]
  • State: [state management approach if detectable]
```

### 4. Understand Data Flow (Grep + Read)

**MUST trace how data flows through the app:**

Use `Grep` to find:
- API calls: `fetch\(`, `axios`, `trpc`, `useQuery`, `useMutation`
- Database queries: `prisma.`, `drizzle`, `sequelize`, `mongoose`
- State management: `useState`, `useContext`, `createStore`, `zustand`
- Server actions: `"use server"`

Then `Read` key files to understand:
- How frontend fetches data
- How backend handles requests
- Database models/relationships

After analysis, output:

```
DATA FLOW
  • Frontend fetches: [method: fetch/axios/trpc]
  • Backend: [framework: Next.js API routes / Express / Fastify]
  • Database: [ORM or raw SQL]
  • Realtime: [websocket/streaming if present]
```

### 5. Style Guide & Patterns (Glob + Read)

**MUST identify coding patterns and conventions:**

Use `Glob` to find:
- Style guide: `**/STYLEGUIDE*`, `**/CONTRIBUTING*`, `**/.prettierrc*`, `**/eslint*`
- Component patterns: Look at 2-3 components to identify patterns

Use `Read` to check:
- Any style guide files
- ESLint/Prettier config
- Example component to identify patterns

After analysis, output:

```
PATTERNS
  • Styling: [Tailwind / CSS modules / styled-components]
  • Components: [functional/class, naming convention]
  • Imports: [absolute/relative paths, alias like @/]
  • Types: [TypeScript strictness, interfaces vs types]
```

### 6. Key Functionality (Grep + Read)

**MUST identify main features by searching for key terms:**

Use `Grep` to find core features:
- Auth: `auth`, `login`, `session`, `clerk`, `next-auth`
- Payments: `stripe`, `payment`, `checkout`
- Upload: `upload`, `s3`, `storage`
- Email: `sendMail`, `resend`, `nodemailer`
- AI: `openai`, `anthropic`, `claude`, `gpt`
- Websockets: `socket`, `websocket`, `pusher`

After analysis, output:

```
FEATURES
  • Auth: [provider if found]
  • Payments: [provider if found]
  • Storage: [provider if found]
  • Email: [provider if found]
  • AI: [provider if found]
  • Other: [any other notable integrations]
```

### 7. Deployment & External Services (Bash)

```bash
echo ""
echo "DEPLOYMENT"

# Check deployment configs
[ -f "vercel.json" ] && echo "  → Vercel"
[ -f "fly.toml" ] && echo "  → Fly.io"
[ -f "Dockerfile" ] && echo "  • Docker"
[ -f "docker-compose.yml" ] && echo "  • Docker Compose"
[ -d "supabase" ] && echo "  • Supabase"
[ -d ".github/workflows" ] && echo "  • GitHub Actions"

# Environment
echo ""
echo "ENVIRONMENT"
[ -f ".env.example" ] && echo "  ✓ .env.example"
[ -f ".env" ] && echo "  ✓ .env (local)"
```

### 8. Summary

After all analysis, output a **concise summary**:

```
SUMMARY
  [1-2 sentences describing what the app does]
  [1-2 sentences on architecture]
  [1 sentence on key tech]

→ complete
```

---

## Output Format

```
→ inspecting my-app...

GIT
  • main @ a1b2c3d (2 days ago)
  → your-username/my-app

TYPE
  • nextjs

[... directory listing ...]

CONFIG
  • Framework: Next.js 14 (App Router)
  • Build: next build
  • Dev: next dev
  • Key deps: @supabase/supabase-js, stripe, resend

ARCHITECTURE
  • Entry: app/layout.tsx
  • Routes: 12 API endpoints
  • Components: 24 components
  • Database: PostgreSQL via Supabase
  • State: React Context + SWR

DATA FLOW
  • Frontend fetches: SWR + fetch
  • Backend: Next.js Route Handlers
  • Database: Supabase client
  • Realtime: Supabase realtime subscriptions

PATTERNS
  • Styling: Tailwind CSS
  • Components: Functional, PascalCase
  • Imports: @/ alias for src/
  • Types: Strict TypeScript, interfaces

FEATURES
  • Auth: Supabase Auth
  • Payments: Stripe
  • Storage: Supabase Storage
  • Email: Resend
  • AI: None
  • Other: Webhooks

DEPLOYMENT
  → Vercel
  • Supabase

ENVIRONMENT
  ✓ .env.example

SUMMARY
  SaaS application with user authentication and subscription billing.
  Uses Next.js App Router with Supabase for database and auth.
  Key tech: Next.js, Supabase, Stripe, Tailwind.

→ complete
```

---

## Key Requirements

- **Must read files**: Agent MUST use Read tool on key files, not just count them
- **Must grep patterns**: Agent MUST search for patterns to understand architecture
- **Must summarize**: End with human-readable summary of what the app does
- **Concise output**: Keep sections brief, not verbose
- **Read-only**: No modifications to any files
