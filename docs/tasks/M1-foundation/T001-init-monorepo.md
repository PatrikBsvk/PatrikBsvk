# T001 — Initialize Next.js monorepo

**Milestone:** M1 — Foundation
**Status:** Ready
**Estimate:** 30–45 min
**Dependencies:** none

---

## Context
Vibell is a guided AI app builder for non-programmers. The repo is currently empty (only `app/.gitkeep` placeholder and the `docs/` tree). We're starting on branch `claude/github-repo-setup-NvGZL`.

We need a clean monorepo foundation that scales to multiple packages: the main web app, the agent system, shared UI, and generated-app templates.

**Reference docs the agent should read first:**
- `docs/plan.md` (vision and roadmap)
- `docs/product/vision.md` (one-liner, success metrics)
- `docs/architecture/overview.md` (target repo layout, tech stack)

## Goal
A working Next.js 15 app inside a pnpm monorepo, with Tailwind 4 and shadcn/ui set up, that builds and runs locally via `pnpm dev`.

## Acceptance Criteria
- [ ] `pnpm-workspace.yaml` at repo root listing `apps/*` and `packages/*`
- [ ] Root `package.json` with workspace scripts (`dev`, `build`, `typecheck`, `lint`)
- [ ] `tsconfig.base.json` at repo root, extended by package configs
- [ ] `apps/web` is a Next.js 15 App Router project (TypeScript, ESLint, Tailwind, src dir, `@/*` alias)
- [ ] **Tailwind CSS 4** configured (uses `@import "tailwindcss"` syntax, NOT v3 config)
- [ ] **shadcn/ui** initialized inside `apps/web`
- [ ] Default shadcn `Button` component added and rendered on `/` to verify
- [ ] `pnpm dev` runs the app at http://localhost:3000
- [ ] `pnpm build` succeeds
- [ ] `pnpm typecheck` (or `pnpm -r typecheck`) passes with strict mode
- [ ] `.gitignore` covers `node_modules`, `.next`, `.env*`, `.DS_Store`, dist artifacts
- [ ] Old `app/.gitkeep` is removed (we're using `apps/web` now)

## Files to create/modify
- `pnpm-workspace.yaml`
- `package.json` (root)
- `tsconfig.base.json`
- `.gitignore`
- `apps/web/**` (full Next.js scaffold)
- DELETE `app/.gitkeep`

## Implementation Notes
- TypeScript strict mode ON.
- Use **pnpm** (not npm/yarn). Confirm pnpm is installed before scaffolding.
- Scaffold command: `pnpm create next-app@latest apps/web --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --turbopack`
- For Tailwind 4: confirm `apps/web/src/app/globals.css` uses `@import "tailwindcss";`. If the scaffold produced v3-style config, migrate to v4.
- shadcn init: `cd apps/web && pnpm dlx shadcn@latest init` (pick Default style, neutral colour, CSS variables yes).
- Add Button: `pnpm dlx shadcn@latest add button`.
- Render the Button on `/` (e.g., `<Button>Hello Vibell</Button>`).
- Do NOT add product features. This is infrastructure only.
- Do NOT create README.md — `docs/` is the documentation source of truth.

## ADR-001 (record while doing this task)
Create `docs/architecture/adrs/ADR-001-monorepo-pnpm.md` with:
- Context: why monorepo, why pnpm
- Decision: pnpm workspaces (vs Turborepo, Nx, single repo)
- Consequences: simpler than Turborepo, fast installs, Vercel supports natively
- Alternatives: Turborepo (added later if caching needed), single repo (rejected — packages will multiply)

## Agent Prompt (copy-paste this into your local Claude Code)

```
You are a senior engineer working on Vibell, a guided AI app builder for non-programmers.

Read these files first:
- docs/plan.md
- docs/product/vision.md
- docs/architecture/overview.md
- docs/tasks/M1-foundation/T001-init-monorepo.md (this task spec — full requirements there)

Your task: T001 — initialize a Next.js 15 monorepo.

Constraints:
- Branch: claude/github-repo-setup-NvGZL (already checked out)
- pnpm workspaces (not Turborepo for now)
- Next.js 15 + React 19 + TypeScript strict + Tailwind 4 + shadcn/ui in apps/web
- Render shadcn Button on / to verify setup
- pnpm dev / build / typecheck must all succeed
- Delete the placeholder app/.gitkeep
- Write ADR-001 documenting the monorepo decision (see task spec)
- Do NOT add any product features in this task
- Do NOT create README.md files

When done:
1. Verify all acceptance criteria from the task file
2. Update the task file to set Status: Done
3. Commit with message: task(T001): init Next.js monorepo with shadcn/ui
4. Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All acceptance criteria checked
- ADR-001 written
- Task file Status updated to **Done**
- Committed with message `task(T001): init Next.js monorepo with shadcn/ui`
- Pushed to `claude/github-repo-setup-NvGZL`
