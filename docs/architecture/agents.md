# Vibell — Agent System

## Philosophy
Specialized agents over one big prompt. Each agent has narrow scope, minimal context, and a specific model. **Smaller prompts = cheaper runs + better outputs.**

## Agent registry

### Orchestrator
- **Model:** Sonnet 4.6
- **Job:** Routes user intent to right agent, plans multi-step flows.
- **Inputs:** User message + project state summary
- **Outputs:** `{ agent: string, payload: object }[]` plan
- **Cache:** System prompt cached (changes rarely)

### Builder
- **Model:** Sonnet 4.6
- **Job:** Generates initial project from wizard output. Picks the right template, fills in content, creates DB schema.
- **Inputs:** Wizard data + template choice
- **Outputs:** File map (`{ path: string, content: string }[]`) for the new project
- **Cache:** System prompt + selected template scaffold cached

### UI Editor
- **Model:** Haiku 4.5
- **Job:** Micro-edits on a single component (text, color, layout, copy change).
- **Inputs:** Component file content + click context (selected element id) + user instruction
- **Outputs:** Diff (or replacement file) for that one component
- **Cache:** System prompt cached
- **Scope:** Always one file. Never sees other components.

### Variant
- **Model:** Haiku 4.5 (3 parallel calls)
- **Job:** Generates 3 alternatives for the same UI change so user picks visually.
- **Inputs:** Same as UI Editor
- **Outputs:** 3 file diffs
- **Cost:** 3× UI Editor, but user gets parallel options instead of guess-and-iterate.

### Data Schema
- **Model:** Sonnet 4.6
- **Job:** Translates user intent ("save users with email and avatar") into Supabase schema + migrations + TypeScript types.
- **Inputs:** Existing schema + user instruction
- **Outputs:** SQL migration + updated TypeScript types
- **Cache:** System prompt + existing schema cached

### Deploy
- **Model:** Haiku 4.5 (mostly deterministic, AI used for slug suggestions and copy)
- **Job:** Pushes generated app to Vercel via REST API, creates subdomain, registers in DB.
- **Inputs:** Project file map + user account
- **Outputs:** Live URL + Vercel project id

### Coach
- **Model:** Haiku 4.5
- **Job:** Conversational helper. Explains what's happening. Onboards new users. Runs the welcome tour.
- **Inputs:** Project state + user question
- **Outputs:** Friendly text + optional action suggestion
- **Cache:** System prompt + Coach personality cached

### Memory
- **Model:** Haiku 4.5
- **Job:** Learns the user. Builds a Profile (cross-project) and per-project Memory. Runs as a background digest. Never user-facing directly.
- **Inputs:** Recent agent_calls + user inputs cache + existing memory
- **Outputs:** Updated facts (add / reinforce / drop)
- **Runs:** Every 10 user actions OR every 15 min of active session
- **User-visible cost:** 0 credits (we absorb — see `docs/design/memory-agent.md`)
- **See:** `docs/design/memory-agent.md` for full spec

### Studio
- **Model:** Sonnet 4.6
- **Job:** Layout-level changes inside Studio Mode (add/remove/reorder sections, edit JSX, custom CSS).
- **Inputs:** Full file map + user instruction + memory context
- **Outputs:** Updated FileMap (or diff)
- **See:** `docs/design/studio-mode.md` for full spec, pricing, plan gating

## Token-saving rules (mandatory)

1. **Prompt caching always on** — system prompts + templates + existing schema all cached.
2. **Component-scoped context** — UI Editor sees one file, never the whole project.
3. **Diff outputs preferred** — agents return diffs, not full files, where the model can.
4. **Cached scaffolds** — common wizard outputs are memoized in DB; if same wizard combo seen before, return cached scaffold.
5. **Cheap-first model** — start with Haiku, escalate to Sonnet only if needed.
6. **Hard timeouts** — every agent call has 30s timeout; runaway calls killed.
7. **Memory injection is free** — Memory Agent's profile + project memory are loaded as cached system blocks for every other agent's call. Cache_read tokens are ~90 % cheaper than fresh input, so personalization comes at near-zero marginal cost.

## Implementation guidelines

- Use `@anthropic-ai/sdk` directly for now (Claude Agent SDK adds overhead we don't need).
- Each agent = a class in `packages/agents/src/<agent-name>.ts` exposing `run(input): Promise<output>`.
- Orchestrator coordinates via simple function calls, NOT Claude tool use (faster, cheaper, deterministic routing).
- All agents log to `agent_calls` table: input tokens, output tokens, cache hits, latency, cost.
- All agents emit events for the credit ledger.

## Telemetry per call (record in DB)

```ts
{
  id: uuid,
  user_id: uuid,
  project_id: uuid | null,
  agent: 'orchestrator' | 'builder' | 'ui_editor' | ...,
  model: 'claude-sonnet-4-6' | 'claude-haiku-4-5-20251001',
  input_tokens: number,
  output_tokens: number,
  cache_read_tokens: number,
  cache_creation_tokens: number,
  latency_ms: number,
  credits_charged: number,
  cost_usd: number,
  created_at: timestamptz,
}
```

This table is the source of truth for unit economics dashboards.
