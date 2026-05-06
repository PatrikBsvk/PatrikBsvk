# T005 — Agent runner: Anthropic SDK, telemetry, credit deduction

**Milestone:** M1 — Foundation
**Status:** Ready
**Estimate:** 2–3 h
**Dependencies:** T002, T003

---

## Context
Every AI feature in Vibell calls Claude via specialized agents (Builder, UI Editor, Coach, etc.). We need shared infrastructure so each agent: (a) calls Claude with prompt caching, (b) logs into `agent_calls` for unit economics, (c) deducts credits atomically, (d) handles errors and timeouts uniformly.

**This task does NOT implement any specific agent's logic — it builds the *runner* that all agents use.** The first concrete agent (Coach as a smoke test) is included.

**Reference docs:**
- `docs/architecture/agents.md` (agent registry, token-saving rules, telemetry shape)
- `docs/architecture/credits.md` (action → credits mapping)

## Goal
A `packages/agents` package exporting `runAgent({ agent, input, userId, projectId? })` that:
- Calls Claude with the right model + cached system prompt
- Records token usage, latency, cost, credits in `agent_calls` and `credit_transactions`
- Throws if user has insufficient credits BEFORE the call
- A working "Coach" agent as a smoke-test endpoint (`POST /api/agents/coach`) that says hi

## Acceptance Criteria
- [ ] `packages/agents/` workspace package created with `package.json`, `tsconfig.json`
- [ ] Exports: `runAgent`, types `AgentName`, `AgentInput`, `AgentOutput`
- [ ] `@anthropic-ai/sdk` dependency installed at this package
- [ ] Prompt-caching enabled via `cache_control` on system messages and shared context
- [ ] Cost calculation per call (input + cache_read + cache_write + output tokens) → USD cents → credits
- [ ] Atomic credit deduction with row-locking SQL (or RPC) — no double-spend possible
- [ ] On failure (API error, timeout 30s): record the call as failed, do NOT charge credits, surface a typed error to caller
- [ ] One concrete agent: `coach` — system prompt instructs Claude to be a friendly Vibell helper; takes `{ message, projectContext? }`; returns `{ reply }`
- [ ] Route handler: `apps/web/src/app/api/agents/coach/route.ts` — auth-checks user, calls `runAgent`, returns reply
- [ ] Smoke test: from logged-in dashboard, a button "Talk to Coach" sends "Hi" → renders Coach reply, records 1+ row in `agent_calls`, deducts credits per `credits.md` (Coach reply = 1 credit per ~200 tokens out)
- [ ] Unit tests for cost calculation and credit math
- [ ] `pnpm typecheck` passes across workspaces

## Files to create
```
packages/agents/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts                    // exports runAgent + types
│   ├── runner.ts                   // the core runAgent function
│   ├── pricing.ts                  // model → USD-per-token map, cost calc
│   ├── credits.ts                  // action → credits mapping (mirrors docs)
│   ├── telemetry.ts                // insert into agent_calls + credit_transactions
│   ├── errors.ts                   // typed error classes
│   └── agents/
│       └── coach.ts                // agent definition: { name, model, systemPrompt, run }
└── tests/
    ├── pricing.test.ts
    └── credits.test.ts

apps/web/src/
├── app/api/agents/coach/route.ts
└── components/dev/coach-smoke-test.tsx  // dashboard widget, dev only
```

## Architecture

### `runAgent` signature
```ts
type AgentName = 'coach' | 'orchestrator' | 'builder' | 'ui_editor' | 'variant' | 'data_schema' | 'deploy';

interface AgentInput {
  // varies per agent
  [key: string]: unknown;
}

interface RunAgentArgs<T = AgentInput> {
  agent: AgentName;
  input: T;
  userId: string;
  projectId?: string;
}

interface RunAgentResult<O = unknown> {
  output: O;
  callId: string;          // agent_calls.id
  creditsCharged: number;
  costUsdCents: number;
  inputTokens: number;
  outputTokens: number;
  cacheReadTokens: number;
  cacheCreationTokens: number;
  latencyMs: number;
}

async function runAgent<I, O>(args: RunAgentArgs<I>): Promise<RunAgentResult<O>>;
```

### Flow inside `runAgent`
1. Look up the agent definition (model, system prompt, action key).
2. Estimate cost cap: read user's `credit_balances`. If `credits_remaining < minimum estimated`, throw `InsufficientCreditsError` (no API call).
3. Call Claude with `cache_control` on system prompt and shared context blocks.
4. Compute exact cost from response usage object.
5. Convert USD cents → credits via mapping (see `pricing.ts` + `credits.ts`).
6. In a single Postgres transaction (or via RPC):
   - Insert into `agent_calls`
   - Insert into `credit_transactions` (negative delta)
   - Update `credit_balances.credits_remaining` atomically
7. Return `RunAgentResult`.

### Pricing table
```ts
// USD per million tokens (2026 rates — verify before launch)
export const MODEL_PRICING = {
  'claude-sonnet-4-6':         { input: 3.00,  cacheRead: 0.30,  cacheWrite: 3.75, output: 15.00 },
  'claude-haiku-4-5-20251001': { input: 1.00,  cacheRead: 0.10,  cacheWrite: 1.25, output: 5.00 },
  'claude-opus-4-7':           { input: 15.00, cacheRead: 1.50,  cacheWrite: 18.75,output: 75.00 },
} as const;
```

> ⚠️ Rates above are placeholders. Pull live rates from Anthropic docs before merging T005.

### Credit mapping
```ts
export const ACTION_CREDIT_COST = {
  new_project: 5,
  ui_edit: 1,
  click_edit: 2,
  variant: 3,
  full_section: 5,
  schema_change: 3,
  voice_wizard: 2,
  coach_message: 1,
} as const;
```

If the actual API cost (USD) maps to MORE credits than the action's flat cost (e.g., user wrote a huge message → many tokens), record both: charge the action's flat credits AND log the actual cost. This is the simplest UX — predictable for users — and we monitor for abuse via the `agent_calls` table.

### Coach agent (concrete first agent)
```ts
export const coachAgent = {
  name: 'coach',
  model: 'claude-haiku-4-5-20251001',
  action: 'coach_message',
  systemPrompt: `You are Vibell Coach, a friendly helper who guides non-programmers through building web apps.
Speak in clear, simple language. Never use emojis. Never use jargon. If a user asks something off-topic, redirect them to building their app.
Keep replies under 80 words unless the user asks for more detail.`,
  // marked as cache_control "ephemeral" so Claude caches it across calls
  run: async (input: { message: string; projectContext?: string }) => {
    // ... build messages, call Claude
  },
} as const;
```

### Atomic credit deduction (Postgres function)
Add a migration `supabase/migrations/<ts>_charge_credits_rpc.sql`:

```sql
create or replace function public.charge_credits(
  p_user_id uuid,
  p_credits int,
  p_action text,
  p_project_id uuid default null,
  p_agent_call_id uuid default null
) returns int
language plpgsql
security definer
as $$
declare
  remaining int;
begin
  -- lock the row
  select credits_remaining into remaining
  from public.credit_balances
  where user_id = p_user_id
  for update;

  if remaining is null then
    raise exception 'NO_BALANCE';
  end if;

  if remaining < p_credits then
    raise exception 'INSUFFICIENT_CREDITS';
  end if;

  update public.credit_balances
  set credits_remaining = remaining - p_credits, updated_at = now()
  where user_id = p_user_id;

  insert into public.credit_transactions (user_id, project_id, action, credits_delta, agent_call_id)
  values (p_user_id, p_project_id, p_action, -p_credits, p_agent_call_id);

  return remaining - p_credits;
end;
$$;
```

The runner calls this RPC inside the same logical transaction as the `agent_calls` insert.

## Implementation Notes
- Use `@anthropic-ai/sdk@^0.40.0` or current; ensure prompt caching is supported.
- Always set `max_tokens` per agent (e.g., Coach = 400; Builder = 8000) to bound cost.
- Log latency, model, tokens, both cache fields, charged credits — these power our internal margin dashboard.
- Wrap Anthropic call with `Promise.race` against a 30s timeout.
- Keep `runAgent` totally agent-agnostic. New agents register a definition object.

## Agent Prompt (copy-paste this)

```
You are a senior engineer working on Vibell.

Read first:
- docs/architecture/agents.md
- docs/architecture/credits.md
- docs/tasks/M1-foundation/T005-agent-runner.md (full spec)

Task T005: build the agent runner package + Coach as the first concrete agent.

Critical points:
- Verify Anthropic SDK pricing rates against current docs before committing pricing.ts.
- Atomic credit deduction via the charge_credits RPC; runner must NOT update balances directly.
- Failed API calls must NOT charge credits.
- 30s timeout on every Claude call.
- Prompt caching enabled on system prompts (cache_control: { type: 'ephemeral' }).

Do this:
1. Create packages/agents workspace package.
2. Implement runner.ts, pricing.ts, credits.ts, telemetry.ts, errors.ts.
3. Add the charge_credits RPC migration and apply it.
4. Implement the Coach agent (one file under src/agents/).
5. Add /api/agents/coach route handler in apps/web that auth-checks the user and calls runAgent.
6. Add a simple dev-only smoke widget on /app/dashboard so we can manually test.
7. Write unit tests for pricing and credit math.
8. Verify pnpm typecheck and pnpm build pass.

When done:
- Update task file Status: Done
- Commit: task(T005): agent runner with Coach + atomic credit deduction
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All criteria checked
- Manual smoke test confirms: Coach reply rendered, `agent_calls` row inserted with sane numbers, balance decremented atomically
- Unit tests green
- Status updated, committed, pushed
