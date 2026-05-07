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

### Brand Designer
- **Model:** Sonnet 4.6 + image generation
- **Job:** Generates a complete brand bible (logo, color system, typography, voice samples) from one description, applies across project.
- **Cost:** ~15–20 credits (TBD)
- **See:** D-004 in `docs/product/decisions-log.md` (design doc pending)

### Extractor (Smart Autofill)
- **Model:** Haiku 4.5 + headless screenshot
- **Job:** Given a URL, extract logo, palette, copy, tone, brand assets and pre-fill the wizard.
- **Cost:** 3 credits
- **See:** D-001

### Debugger
- **Model:** Sonnet 4.6
- **Job:** Reads build/preview/publish errors → proposes fix → optionally applies.
- **Cost:** **0 credits** for user (Vibell absorbs — retention play)
- **See:** D-002

### Integrations
- **Model:** Sonnet 4.6
- **Job:** Wires up third-party integrations (Stripe, Calendly, Mailchimp, …) in user apps. Validates API keys, scaffolds components, runs smoke tests.
- **Cost:** 5–10 credits per integration setup (per integration spec)
- **See:** D-003

### Analytics
- **Model:** Haiku 4.5
- **Job:** Audits published apps' real metrics, suggests UI/copy improvements based on data.
- **Cost:** 0 credits for tracking, 3 credits per AI audit
- **See:** D-006

### SEO
- **Model:** Haiku 4.5
- **Job:** Audits published apps for SEO, suggests one-click fixes (meta, alt text, schema).
- **Cost:** 5 credits per audit
- **See:** D-008

### Image
- **Model:** External image API (Anthropic image gen or Flux)
- **Job:** Generates branded hero images, illustrations, mocks, locked to project palette/tone.
- **Cost:** 8 credits per image (paid by user)
- **See:** D-009

### Continuity (Coach extension)
- **Model:** Haiku 4.5
- **Job:** On dashboard / project return, proactively suggests next steps based on project state + Memory.
- **Cost:** 0 credits (background)
- **See:** D-010

### Voice
- **Model:** Speech-to-text (Whisper-class) → existing agents (Coach, UI Editor, etc.)
- **Job:** Transcribe user voice across the workspace; pipe to the appropriate agent.
- **Cost:** 0 credits for user (Vibell absorbs ~$0.005/min)
- **Plan:** available on **all plans** (founder direction, D-007)
- **See:** D-007

### Creator Hub Membership (D-012 confirmed)
- **Model:** Sonnet 4.6
- **Job:** Sets up creator's own paid membership on their Vibell profile (`vibell.app/@username`). Wires Stripe Connect, creates tiers, gates articles/tips by tier, manages follower roles.
- **Cost:** 10 credits per setup. Vibell takes 7–15 % platform fee on creator's membership MRR (per plan tier).
- **See:** D-012 in decisions-log

### Article Writer (assists creator hub content)
- **Model:** Sonnet 4.6
- **Job:** Drafts long-form articles for the Creator Hub from a topic + project context + memory. Optional, the creator can write from scratch.
- **Cost:** 2 credits per draft
- **See:** D-012 (Creator Hub)

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
