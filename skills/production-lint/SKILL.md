---
name: production-lint
description: Run before Vercel deployment. Catches type errors, lint issues, build failures, and unused dependencies.
category: deployment
priority: 9
---

# Production Lint

Run before Vercel deployment. Catches type errors, lint issues, build failures, and unused dependencies.

## Usage

Invoke this skill before deploying to Vercel or when CI fails.

---

## Lint Sequence

Run these commands **in order**. Stop and fix on failure.

### 1. TypeScript Type Check

```bash
npx tsc --noEmit
```

**Common failures:**
- Type mismatch → Fix the type or add proper typing
- Missing types → Install `@types/package` or create declaration

---

### 2. ESLint

```bash
npm run lint
```

**Common failures:**
- Unused variable → Remove it or use `_` prefix
- Missing dependency → Add to useEffect array
- Broken import → Fix path or create file

---

### 3. Production Build

```bash
npm run build
```

**Common failures:**
- Missing env var → Check `.env.local` has required vars
- Import error → File doesn't exist or wrong path
- Route conflict → Duplicate page/route files

---

### 4. Unused Dependencies

```bash
npx depcheck --ignores="@types/*,eslint-config-next"
```

**Output:**
- Unused deps → Remove from package.json if truly unused
- Missing deps → Add to package.json

---

## All-in-One Command

```bash
npx tsc --noEmit && npm run lint && npm run build && npx depcheck --ignores="@types/*,eslint-config-next"
```

All four must pass for green deployment.

---

## Quick Debug Guide

| Error Type | Fix |
|------------|-----|
| `Cannot find module '@/...'` | Check file exists, verify `tsconfig.json` paths |
| `Type 'X' is not assignable to 'Y'` | Fix the type or cast if intentional |
| `Unused variable` | Remove or prefix with `_` |
| `npm ERR! missing script: lint` | Add `"lint": "next lint"` to package.json |
| Build hangs | Check for circular imports or infinite loops |
| Depcheck false positive | Add to `--ignores` list |
