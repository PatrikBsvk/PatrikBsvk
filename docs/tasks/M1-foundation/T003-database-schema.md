# T003 — Initial database schema + RLS

**Milestone:** M1 — Foundation
**Status:** Ready
**Estimate:** 1–1.5 h
**Dependencies:** T002

---

## Context
We need the foundational tables for Vibell: profiles, plans, subscriptions, credit balances and ledger, projects, snapshots, agent calls. RLS policies must isolate user data.

**Reference docs:**
- `docs/architecture/credits.md` (plans, subscriptions, credits tables)
- `docs/architecture/agents.md` (`agent_calls` telemetry table)

## Goal
A migration applied to Supabase that creates all foundational tables with RLS policies, plus seed data for `plans` and TypeScript types generated for the app.

## Acceptance Criteria
- [ ] Migration file in `supabase/migrations/<timestamp>_initial_schema.sql`
- [ ] Tables created: `profiles`, `plans`, `subscriptions`, `credit_balances`, `credit_transactions`, `credit_packs`, `projects`, `project_snapshots`, `agent_calls`
- [ ] RLS enabled on all user-facing tables
- [ ] RLS policies: users see/edit only their own rows; `plans` is public read
- [ ] Trigger: when a new auth.user is created, automatically insert a `profiles` row + `subscriptions` row with plan='free' + `credit_balances` row with 50 credits
- [ ] Seed data for `plans` table (free, basic, standard, pro, team — see `docs/architecture/credits.md`)
- [ ] `pnpm supabase:types` script generates TS types into `apps/web/src/lib/supabase/types.gen.ts`
- [ ] Migration runs successfully against the live Supabase project
- [ ] Smoke test: server-side query `select * from plans` from a route returns 5 rows

## Files to create
- `supabase/migrations/<timestamp>_initial_schema.sql`
- `supabase/seed.sql` (plans seed)
- `apps/web/src/lib/supabase/types.gen.ts` (auto-generated)
- Update root `package.json` with `supabase:types` and `supabase:migrate` scripts
- `apps/web/src/app/api/health/route.ts` — smoke test endpoint that returns plans count

## Schema (full)

```sql
-- =====================================================
-- profiles: extends auth.users with public profile data
-- =====================================================
create table public.profiles (
  id            uuid primary key references auth.users on delete cascade,
  username      text unique,
  display_name  text,
  avatar_url    text,
  bio           text,
  is_creator    boolean not null default false,
  stripe_connect_id text,                  -- for marketplace payouts
  created_at    timestamptz not null default now(),
  updated_at    timestamptz not null default now()
);
alter table public.profiles enable row level security;
create policy "profiles read public"   on public.profiles for select using (true);
create policy "profiles update own"    on public.profiles for update using (auth.uid() = id);

-- =====================================================
-- plans: subscription tiers (static, seeded)
-- =====================================================
create table public.plans (
  id              text primary key,        -- 'free' | 'basic' | 'standard' | 'pro' | 'team'
  display_name    text not null,
  price_usd_cents int  not null,
  monthly_credits int  not null,
  features        jsonb not null,          -- { watermark, custom_domain, marketplace_split }
  stripe_price_id text,
  sort_order      int  not null default 0,
  created_at      timestamptz not null default now()
);
alter table public.plans enable row level security;
create policy "plans read public" on public.plans for select using (true);

-- =====================================================
-- subscriptions: one per user, current plan
-- =====================================================
create table public.subscriptions (
  user_id              uuid primary key references auth.users on delete cascade,
  plan_id              text not null references public.plans,
  stripe_subscription  text,
  status               text not null default 'active',  -- 'active' | 'past_due' | 'canceled'
  current_period_end   timestamptz,
  cancel_at_period_end boolean not null default false,
  created_at           timestamptz not null default now(),
  updated_at           timestamptz not null default now()
);
alter table public.subscriptions enable row level security;
create policy "subscriptions read own"   on public.subscriptions for select using (auth.uid() = user_id);
-- Only service_role writes subscriptions (via webhooks)

-- =====================================================
-- credit_balances: refreshed monthly
-- =====================================================
create table public.credit_balances (
  user_id           uuid primary key references auth.users on delete cascade,
  credits_remaining int  not null,
  period_start      timestamptz not null,
  period_end        timestamptz not null,
  updated_at        timestamptz not null default now()
);
alter table public.credit_balances enable row level security;
create policy "credit_balances read own" on public.credit_balances for select using (auth.uid() = user_id);
-- Only service_role writes balances

-- =====================================================
-- credit_transactions: append-only ledger
-- =====================================================
create table public.credit_transactions (
  id            bigserial primary key,
  user_id       uuid not null references auth.users on delete cascade,
  project_id    uuid,
  action        text not null,            -- 'new_project' | 'ui_edit' | 'variant' | 'monthly_grant' | 'overage'
  credits_delta int  not null,
  agent_call_id uuid,
  created_at    timestamptz not null default now()
);
alter table public.credit_transactions enable row level security;
create policy "credit_transactions read own" on public.credit_transactions for select using (auth.uid() = user_id);
create index credit_transactions_user_created_idx on public.credit_transactions (user_id, created_at desc);

-- =====================================================
-- credit_packs: overage purchases
-- =====================================================
create table public.credit_packs (
  id              bigserial primary key,
  user_id         uuid not null references auth.users on delete cascade,
  credits         int  not null,
  amount_usd_cents int not null,
  stripe_payment  text,
  created_at      timestamptz not null default now()
);
alter table public.credit_packs enable row level security;
create policy "credit_packs read own" on public.credit_packs for select using (auth.uid() = user_id);

-- =====================================================
-- projects: one user-built app
-- =====================================================
create table public.projects (
  id            uuid primary key default gen_random_uuid(),
  owner_id      uuid not null references auth.users on delete cascade,
  slug          text unique not null,
  name          text not null,
  description   text,
  template_id   text,                     -- which template was used to seed
  status        text not null default 'draft',  -- 'draft' | 'published' | 'archived'
  vercel_project_id text,
  live_url      text,
  is_marketplace_listed boolean not null default false,
  marketplace_price_usd_cents int,
  created_at    timestamptz not null default now(),
  updated_at    timestamptz not null default now()
);
alter table public.projects enable row level security;
create policy "projects read own"        on public.projects for select using (auth.uid() = owner_id);
create policy "projects read public listed" on public.projects for select using (is_marketplace_listed = true);
create policy "projects write own"       on public.projects for all using (auth.uid() = owner_id);
create index projects_owner_idx on public.projects (owner_id, created_at desc);

-- =====================================================
-- project_snapshots: auto-save versioning
-- =====================================================
create table public.project_snapshots (
  id          uuid primary key default gen_random_uuid(),
  project_id  uuid not null references public.projects on delete cascade,
  files       jsonb not null,             -- { "path": "content", ... }
  message     text,                       -- e.g. "Changed hero color to gold"
  created_at  timestamptz not null default now()
);
alter table public.project_snapshots enable row level security;
create policy "snapshots read own" on public.project_snapshots
  for select using (
    exists (select 1 from public.projects p where p.id = project_id and p.owner_id = auth.uid())
  );
create index snapshots_project_idx on public.project_snapshots (project_id, created_at desc);

-- =====================================================
-- agent_calls: telemetry for unit economics
-- =====================================================
create table public.agent_calls (
  id                    uuid primary key default gen_random_uuid(),
  user_id               uuid not null references auth.users on delete cascade,
  project_id            uuid references public.projects on delete set null,
  agent                 text not null,    -- 'orchestrator' | 'builder' | ...
  model                 text not null,    -- 'claude-sonnet-4-6' | 'claude-haiku-4-5-20251001'
  input_tokens          int  not null,
  output_tokens         int  not null,
  cache_read_tokens     int  not null default 0,
  cache_creation_tokens int  not null default 0,
  latency_ms            int  not null,
  credits_charged       int  not null,
  cost_usd_cents        int  not null,
  created_at            timestamptz not null default now()
);
alter table public.agent_calls enable row level security;
create policy "agent_calls read own" on public.agent_calls for select using (auth.uid() = user_id);
create index agent_calls_user_created_idx on public.agent_calls (user_id, created_at desc);

-- =====================================================
-- Trigger: bootstrap new users
-- =====================================================
create or replace function public.handle_new_user()
returns trigger
language plpgsql
security definer
set search_path = public
as $$
begin
  insert into public.profiles (id, display_name)
  values (new.id, coalesce(new.raw_user_meta_data->>'display_name', split_part(new.email, '@', 1)));

  insert into public.subscriptions (user_id, plan_id, status)
  values (new.id, 'free', 'active');

  insert into public.credit_balances (user_id, credits_remaining, period_start, period_end)
  values (
    new.id,
    50,
    date_trunc('month', now()),
    date_trunc('month', now()) + interval '1 month'
  );

  insert into public.credit_transactions (user_id, action, credits_delta)
  values (new.id, 'monthly_grant', 50);

  return new;
end;
$$;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute function public.handle_new_user();

-- =====================================================
-- updated_at trigger helper
-- =====================================================
create or replace function public.touch_updated_at()
returns trigger language plpgsql as $$
begin
  new.updated_at = now();
  return new;
end;
$$;

create trigger profiles_touch       before update on public.profiles      for each row execute function public.touch_updated_at();
create trigger subscriptions_touch  before update on public.subscriptions for each row execute function public.touch_updated_at();
create trigger projects_touch       before update on public.projects      for each row execute function public.touch_updated_at();
```

## Seed (`supabase/seed.sql`)

```sql
insert into public.plans (id, display_name, price_usd_cents, monthly_credits, features, sort_order) values
  ('free',     'Free',     0,     50,  '{"watermark": true,  "custom_domain": false, "marketplace_split": 70, "max_projects": 1}'::jsonb,        1),
  ('basic',    'Basic',    1500,  200, '{"watermark": false, "custom_domain": false, "marketplace_split": 70, "max_projects": 5}'::jsonb,        2),
  ('standard', 'Standard', 3900,  1000,'{"watermark": false, "custom_domain": false, "marketplace_split": 70, "max_projects": 25}'::jsonb,       3),
  ('pro',      'Pro',      9900,  3000,'{"watermark": false, "custom_domain": true,  "marketplace_split": 80, "max_projects": null}'::jsonb,    4),
  ('team',     'Team',     25900, 9000,'{"watermark": false, "custom_domain": true,  "marketplace_split": 80, "max_projects": null, "seats": 5}'::jsonb, 5)
on conflict (id) do update set
  display_name = excluded.display_name,
  price_usd_cents = excluded.price_usd_cents,
  monthly_credits = excluded.monthly_credits,
  features = excluded.features,
  sort_order = excluded.sort_order;
```

## Implementation Notes
- All tables in `public` schema. Do NOT modify `auth` schema.
- The `handle_new_user` trigger runs as `security definer` so it can insert across tables. Test by creating a new user and verifying profile/subscription/balance rows appear.
- Use `pnpm dlx supabase db push` (linked project) or `supabase migration up` (local).
- Generate types: `pnpm dlx supabase gen types typescript --project-id <id> > apps/web/src/lib/supabase/types.gen.ts`. Add as `supabase:types` script.
- Smoke endpoint `/api/health` does a server-side query `from('plans').select('id', { count: 'exact', head: true })`.

## Agent Prompt (copy-paste this)

```
You are a senior engineer working on Vibell.

Read first:
- docs/architecture/credits.md
- docs/architecture/agents.md
- docs/tasks/M1-foundation/T003-database-schema.md (full spec, including the SQL)

Task T003: create the initial database schema in Supabase.

Do this:
1. Create migration supabase/migrations/<timestamp>_initial_schema.sql with the SQL from the task spec.
2. Create supabase/seed.sql with the plans seed.
3. Apply the migration to the linked Supabase project (use `supabase db push` or `supabase migration up` per CLI version).
4. Run the seed.
5. Add `supabase:types` and `supabase:migrate` scripts to root package.json.
6. Generate TS types into apps/web/src/lib/supabase/types.gen.ts.
7. Add /api/health route that returns { ok, plansCount } as smoke test.
8. Verify: create a test user via Supabase Studio and confirm profiles/subscriptions/credit_balances rows are auto-inserted by the trigger.

When done:
- Update task file Status: Done
- Commit: task(T003): initial database schema with RLS and seed plans
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- Migration applied to live Supabase project
- Plans seeded, query returns 5 rows
- New-user trigger verified (test user gets bootstrap rows)
- TS types generated and committed
- /api/health returns 200 with plansCount === 5
- Status updated, committed, pushed
