# T002 — Supabase project setup + client wiring

**Milestone:** M1 — Foundation
**Status:** Ready (after manual Supabase project creation)
**Estimate:** 30 min coding (+ 10 min manual setup)
**Dependencies:** T001

---

## Context
Vibell uses Supabase for auth, Postgres database, and storage. We need Supabase wired into the Next.js app so subsequent tasks (T003 schema, T004 auth UI) can use it.

**Reference docs:**
- `docs/architecture/overview.md` (tech stack)
- `docs/architecture/credits.md` (DB tables coming in T003)

## Manual setup (founder must do BEFORE running this task)

1. Go to https://supabase.com → create new project named `vibell-prod` (region closest to target users — for EU/CZ: `eu-central-1`).
2. Wait ~2 min for provisioning.
3. From project Settings → API, copy:
   - **Project URL** (`SUPABASE_URL`)
   - **anon public key** (`SUPABASE_ANON_KEY`)
   - **service_role secret key** (`SUPABASE_SERVICE_ROLE_KEY`)
4. Save these into `apps/web/.env.local` (file should NOT be committed):
   ```
   NEXT_PUBLIC_SUPABASE_URL=...
   NEXT_PUBLIC_SUPABASE_ANON_KEY=...
   SUPABASE_SERVICE_ROLE_KEY=...
   ```
5. Confirm `.gitignore` ignores `.env.local`.

For local dev DB later (optional), the agent will set up `supabase` CLI.

## Goal
Working Supabase client (server-side and client-side) inside `apps/web`, plus the `supabase` CLI configured for migrations.

## Acceptance Criteria
- [ ] `@supabase/supabase-js` and `@supabase/ssr` installed in `apps/web`
- [ ] `supabase` CLI installed as dev dependency (or globally documented)
- [ ] `apps/web/src/lib/supabase/client.ts` — browser client helper
- [ ] `apps/web/src/lib/supabase/server.ts` — server client helper (uses `cookies()` from Next.js)
- [ ] `apps/web/src/lib/supabase/admin.ts` — admin client (service role, server-only, never imported by client code)
- [ ] `apps/web/src/lib/env.ts` — typed env validation using `zod` (fails fast if env vars missing)
- [ ] `apps/web/.env.example` documenting required env vars (committed; real values in `.env.local`, not committed)
- [ ] `supabase` directory at repo root with `config.toml` for the CLI (can run `supabase init`)
- [ ] Smoke test: a server component on `/` reads `auth.getUser()` and shows "logged out" (no real auth yet)
- [ ] `pnpm typecheck` and `pnpm build` still pass

## Files to create
- `apps/web/src/lib/supabase/client.ts`
- `apps/web/src/lib/supabase/server.ts`
- `apps/web/src/lib/supabase/admin.ts`
- `apps/web/src/lib/env.ts`
- `apps/web/.env.example`
- `supabase/config.toml` (via `supabase init`)
- Update `apps/web/src/app/page.tsx` for smoke test

## Implementation Notes
- Use `@supabase/ssr` (modern way for Next.js App Router), not the deprecated `auth-helpers`.
- Server client must read/write cookies via Next.js `cookies()` helper.
- Admin client takes `SUPABASE_SERVICE_ROLE_KEY`; mark file with `import 'server-only'` to prevent client bundle leak.
- Env validation in `env.ts` using `zod`:
  ```ts
  import { z } from 'zod'
  export const env = z.object({
    NEXT_PUBLIC_SUPABASE_URL: z.string().url(),
    NEXT_PUBLIC_SUPABASE_ANON_KEY: z.string().min(1),
    SUPABASE_SERVICE_ROLE_KEY: z.string().min(1),
  }).parse(process.env)
  ```
- Smoke test: in `page.tsx` (server component) call `supabase.auth.getUser()` and render "Signed in as X" or "Signed out".

## ADR-002 (start drafting in this task)
Create `docs/architecture/adrs/ADR-002-supabase.md` — note that Vibell's own DB (auth, projects) is one Supabase project. Per-app DB strategy (one Supabase project per generated app vs single shared with RLS) is a SEPARATE decision deferred to M4.

## Agent Prompt (copy-paste this into your local Claude Code)

```
You are a senior engineer working on Vibell.

Read first:
- docs/plan.md
- docs/architecture/overview.md
- docs/architecture/credits.md
- docs/tasks/M1-foundation/T002-supabase-setup.md (full spec)

Task T002: wire up Supabase in apps/web.

Pre-condition: founder has created the Supabase project and added credentials to apps/web/.env.local. If env vars are missing, FAIL FAST with a clear error message — do not proceed silently.

Do this:
1. Install @supabase/supabase-js, @supabase/ssr, zod, and supabase CLI as dev dep.
2. Create the three Supabase client helpers (client, server, admin) using @supabase/ssr conventions.
3. Add typed env validation in src/lib/env.ts using zod.
4. Run `supabase init` at repo root to create supabase/config.toml.
5. Create apps/web/.env.example with placeholder values.
6. Add a smoke test in src/app/page.tsx — server component calls supabase.auth.getUser() and renders "Signed out" (no real auth UI yet, that's T004).
7. Verify pnpm typecheck and pnpm build still pass.
8. Write ADR-002 documenting the decision to use Supabase for Vibell's own data (per-app DB strategy is deferred).

When done:
- Update task file Status: Done
- Commit: task(T002): wire up Supabase clients and env validation
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All acceptance criteria checked
- ADR-002 written
- Status updated to **Done** in this file
- Committed with `task(T002): wire up Supabase clients and env validation`
- Pushed
