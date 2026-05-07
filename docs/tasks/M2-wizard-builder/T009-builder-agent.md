# T009 — Builder Agent (wizard payload → project file map)

**Milestone:** M2 — Wizard + Builder
**Status:** Ready
**Estimate:** 5–7 h
**Dependencies:** T005 (agent runner), T007 (wizard UI), T008 (templates package)

---

## Context
Builder Agent is the **most critical AI task in MVP**. It takes the structured wizard payload, optionally enriched by user memory and a seed mockup, and produces a working project. Because the layout is locked (Smart Mode), Builder only writes a single typed file: `src/lib/site-config.ts`. Everything else comes from the template scaffold.

**Reference docs (read in order):**
- `/asistenta.md` (founder profile — READ FIRST)
- `docs/architecture/agents.md`
- `docs/architecture/credits.md`
- `docs/design/templates.md`
- `docs/design/memory-agent.md`
- `docs/design/wizard.md`

## Goal
After the user submits the wizard, the Builder Agent runs:
1. Loads the chosen template scaffold via `packages/templates`
2. Reads user Profile + Project memory (if memory consent granted)
3. Generates a typed `siteConfig` object that matches the template's section schema
4. Validates output against zod schema; retries once on validation failure
5. Persists the resulting FileMap as `project_snapshots.files` (`message='wizard:initial'`)
6. Charges 5 credits atomically via `charge_credits` RPC
7. Writes the first batch of project memory facts (Memory Agent post-step)
8. Updates project status to `'draft'` and returns `projectId` for redirect

## Acceptance Criteria
- [ ] New agent registered in `packages/agents`: `src/agents/builder.ts` with definition (model: `claude-sonnet-4-6`, action: `new_project`)
- [ ] System prompt is **cached** via `cache_control: { type: 'ephemeral' }`
- [ ] Memory context loaded from `user_profiles.memory` + `project_memories.memory` and injected as a separate cached system block
- [ ] Output is a strict typed object validated by `zod` matching the template's section schema (different per template)
- [ ] Retry policy: on first zod validation failure, retry **once** with the validation error appended to the prompt
- [ ] On second failure: surface a smart error to the workspace and **do NOT charge credits** (failed-call rule from T005)
- [ ] Successful run writes:
  - `project_snapshots` row with `files` jsonb (the full FileMap from `scaffold(payload)` with the generated `site-config.ts` overlaid)
  - `agent_calls` row with token usage + credits + cost
  - `credit_transactions` row (5 credits negative)
  - `project_memories` row with initial facts distilled from wizard inputs (only if user consented to memory)
- [ ] `/api/projects/draft` route handler from T007 is **upgraded** to call Builder Agent (replaces the stub)
- [ ] Stream progress events to the client: `started → caching → generating → validating → saving → done` so the workspace loading screen has live ETA
- [ ] Time budget: **≤ 60 s** end-to-end p95
- [ ] All token usage, cache hits, and latency logged to `agent_calls`
- [ ] Unit tests: zod validation, retry logic, credit math edge cases, memory injection format
- [ ] Integration test: end-to-end wizard payload → snapshot persisted (uses Supabase test project)

## Files to create / modify

### New
```
packages/agents/src/agents/builder.ts
packages/agents/src/memory/loader.ts        // loads + formats memory blocks
packages/agents/src/memory/post-wizard.ts   // distills wizard inputs into project memory
packages/agents/src/prompts/builder.system.ts
packages/agents/src/prompts/builder.format-memory.ts
packages/agents/tests/builder.test.ts
packages/agents/tests/memory-loader.test.ts

apps/web/src/app/api/agents/builder/route.ts        // primary endpoint (server action style)
apps/web/src/components/workspace/BuildingState.tsx // animated loader for the build phase
```

### Modified
```
apps/web/src/app/api/projects/draft/route.ts    // now calls Builder Agent instead of stub insert
packages/agents/src/runner.ts                   // accepts memoryContext param
packages/agents/src/agents/index.ts             // registers Builder
supabase/migrations/<ts>_user_profiles_and_project_memories.sql  // creates the two memory tables (if not yet from a prior task)
```

## System prompt outline (Builder)

Saved in `packages/agents/src/prompts/builder.system.ts` and cached:

```
You are Vibell's Builder Agent.

Your job: Given a Wizard payload and a chosen template, produce a single TypeScript object — siteConfig — that fills in the content for every section in the template.

Hard rules (ALWAYS):
1. Output ONLY the siteConfig object as a TypeScript "as const" literal. No prose, no comments, no JSX.
2. The structure must match exactly the template's SECTION_SCHEMA (provided below per request).
3. Use Lucide icon names from the ALLOWED_ICONS list (provided per request). Never invent icon names.
4. Tone of generated copy follows USER_PROFILE if present, otherwise the template's default tone for the chosen style preset.
5. Never mention emojis. Never use exclamation marks unless absolutely natural to the brand.
6. Keep copy concise: hero subtitle ≤ 140 chars, FAQ answers ≤ 200 chars, feature body ≤ 100 chars.
7. If a section is not in the wizard's enabled features, set its `enabled: false` and leave content as defaults.
8. Use the user's primary color from the wizard if present; otherwise the brand violet `#7C3AED`.

You will receive (in order, each as a separate cached system block):
- USER_PROFILE memory (style preferences, phrasings, references)
- PROJECT_MEMORY (audience, tone, references for this specific project)
- TEMPLATE_SECTION_SCHEMA (the exact zod-shaped object you must match)
- ALLOWED_ICONS list
- The wizard payload as the user message

Return strictly the TypeScript literal. The runner will parse and validate it.
```

## Memory injection format (`packages/agents/src/memory/loader.ts`)

```ts
export async function loadMemoryContext(
  supabaseAdmin: SupabaseClient,
  userId: string,
  projectId?: string
): Promise<MemoryContext> {
  const [profile, project] = await Promise.all([
    supabaseAdmin
      .from('user_profiles')
      .select('memory, paused')
      .eq('user_id', userId)
      .maybeSingle(),
    projectId
      ? supabaseAdmin
          .from('project_memories')
          .select('memory')
          .eq('project_id', projectId)
          .maybeSingle()
      : Promise.resolve({ data: null }),
  ]);

  if (profile.data?.paused) {
    return { profile: '', project: '' };
  }

  return {
    profile: formatMemoryForPrompt('USER_PROFILE', profile.data?.memory),
    project: formatMemoryForPrompt('PROJECT_MEMORY', project.data?.memory),
  };
}

export function formatMemoryForPrompt(label: string, memory: MemoryJson | null): string {
  if (!memory || !memory.facts || memory.facts.length === 0) return '';
  const facts = memory.facts
    .filter((f) => f.confidence >= 0.4)         // skip low-confidence noise
    .sort((a, b) => b.confidence - a.confidence)
    .slice(0, 25)                                // keep prompt small
    .map((f) => `- [${f.key}] ${f.value} (confidence ${f.confidence.toFixed(2)})`)
    .join('\n');
  return `# ${label}\n${facts}\n`;
}
```

## Builder run signature

```ts
interface BuilderInput {
  wizardPayload: WizardPayload;
  templateManifest: TemplateManifest;
  userId: string;
  projectId: string;
}

interface BuilderOutput {
  siteConfig: unknown;          // validated against template's zod schema after parse
  scaffoldFiles: FileMap;       // from template.scaffold(payload)
  finalFileMap: FileMap;        // scaffoldFiles with siteConfig overlaid
}

export const builderAgent: Agent<BuilderInput, BuilderOutput> = {
  name: 'builder',
  model: 'claude-sonnet-4-6',
  action: 'new_project',         // 5 credits per credits.md
  maxTokens: 8000,
  timeoutMs: 60_000,
  systemPrompt: BUILDER_SYSTEM,
  run: async (input, { client, memory }) => {
    const sectionSchema = input.templateManifest.sectionSchema; // zod schema export
    const allowedIcons = input.templateManifest.allowedIcons;

    const messages = [
      { role: 'system', content: BUILDER_SYSTEM, cache_control: { type: 'ephemeral' } },
      memory.profile && { role: 'system', content: memory.profile, cache_control: { type: 'ephemeral' } },
      memory.project && { role: 'system', content: memory.project, cache_control: { type: 'ephemeral' } },
      { role: 'system', content: `TEMPLATE_SECTION_SCHEMA:\n${sectionSchema.toString()}\n\nALLOWED_ICONS:\n${allowedIcons.join(', ')}` },
      { role: 'user', content: JSON.stringify(input.wizardPayload, null, 2) },
    ].filter(Boolean);

    let response = await client.messages.create({ model: 'claude-sonnet-4-6', messages, max_tokens: 8000 });
    let parsed = tryParseTsLiteral(response.content[0].text);
    let validation = sectionSchema.safeParse(parsed);

    if (!validation.success) {
      const retryMessages = [...messages,
        { role: 'assistant', content: response.content[0].text },
        { role: 'user', content: `Validation failed: ${JSON.stringify(validation.error.issues)}\nReturn ONLY the corrected siteConfig literal.` },
      ];
      response = await client.messages.create({ model: 'claude-sonnet-4-6', messages: retryMessages, max_tokens: 8000 });
      parsed = tryParseTsLiteral(response.content[0].text);
      validation = sectionSchema.safeParse(parsed);
      if (!validation.success) throw new BuilderValidationError(validation.error);
    }

    const scaffoldFiles = await input.templateManifest.scaffold(input.wizardPayload);
    const finalFileMap = {
      ...scaffoldFiles,
      'src/lib/site-config.ts': renderSiteConfigTs(validation.data),
    };

    return { siteConfig: validation.data, scaffoldFiles, finalFileMap };
  },
};
```

`tryParseTsLiteral` is a tiny utility that accepts an `as const` TS object literal and returns a JS value (use `eval` in a sandboxed VM context, or a JSON5-friendly parser like `json5` after stripping the `as const` suffix).

`renderSiteConfigTs(obj)` produces the file content:

```ts
export const siteConfig = ${JSON.stringify(obj, null, 2)} as const;
```

## Post-wizard memory step

After Builder succeeds, fire-and-forget the Memory Agent's `post-wizard.ts`:

```ts
export async function postWizardMemoryWrite({
  userId,
  projectId,
  wizardPayload,
  hasConsent,
  client,
  supabaseAdmin,
}): Promise<void> {
  if (!hasConsent) return;

  // tiny Haiku call — distill 3-5 project facts from wizard inputs
  const facts = await distillProjectFactsFromWizard(client, wizardPayload);
  await supabaseAdmin.from('project_memories').upsert({
    project_id: projectId,
    memory: { version: 1, facts },
    updated_at: new Date().toISOString(),
  });
}
```

This runs **outside** the Builder credit charge — it's free (we absorb).

## Streaming progress events

Use Server-Sent Events from the route handler:

```
event: started
data: {}

event: caching
data: { stage: "loading_memory" }

event: generating
data: { stage: "claude_call", etaSeconds: 30 }

event: validating
data: {}

event: saving
data: {}

event: done
data: { projectId: "...", liveCallId: "..." }
```

Workspace loader subscribes via EventSource and animates the stages.

## Smart errors

If validation retry fails or Claude times out, return a structured error to the workspace:

```json
{
  "error": "BUILDER_VALIDATION_FAILED",
  "userMessage": "We couldn't finish that. Often a template change fixes it.",
  "actions": [
    { "label": "Retry", "action": "retry" },
    { "label": "Try a different template", "action": "back_to_wizard_step_1" }
  ]
}
```

The workspace renders this as the smart-error UI per `docs/design/ux-flows.md` section G.

## Implementation Notes
- **Strict TypeScript everywhere.** No `any`.
- Use `vm.runInNewContext` (Node) or a small custom evaluator for `as const` literals — never raw `eval`.
- Section schemas live next to each template manifest (e.g., `packages/templates/src/landing/schema.ts` exports `landingSectionSchema: z.ZodSchema`).
- Allowed-icons whitelist is per-template (max ~30 icons each) — keeps the model's choices bounded.
- Builder must NEVER write any layout TSX. The runner enforces this by only ever overwriting `src/lib/site-config.ts` in the FileMap.
- Memory tables are created by a small migration in this task — schedule the migration before the Builder code runs in tests.

## Agent Prompt (copy-paste this)

```
You are a senior engineer working on Vibell.

READ FIRST (in order):
- /asistenta.md (founder profile — non-negotiable rules)
- docs/architecture/agents.md
- docs/architecture/credits.md
- docs/design/templates.md
- docs/design/memory-agent.md
- docs/design/wizard.md
- docs/tasks/M2-wizard-builder/T009-builder-agent.md (this task)

Task T009: implement the Builder Agent.

Hard rules:
- Builder writes ONLY src/lib/site-config.ts. The runner must enforce this; reject any other file in the model output.
- Strict zod validation against per-template section schema. Retry once on failure, then surface a smart error.
- Memory injection: load user_profiles.memory + project_memories.memory; format and pass as cached system blocks. If user paused memory, skip.
- Atomic 5-credit charge via charge_credits RPC; failed calls do NOT charge.
- Stream progress via SSE so the workspace loader can show live stages.
- 60-second p95 time budget end to end.
- After success, fire-and-forget the post-wizard memory write (free for the user).

Do this:
1. Add the user_profiles + project_memories migration if not yet present.
2. Implement packages/agents/src/agents/builder.ts and the supporting memory/loader.ts and memory/post-wizard.ts.
3. Implement the SSE-streaming /api/agents/builder route handler.
4. Replace the stub /api/projects/draft handler from T007 with one that calls Builder.
5. Add unit tests + an integration test against a Supabase test project.
6. Verify pnpm typecheck and pnpm build pass across workspaces.

When done:
- Update task file Status: Done
- Commit: task(T009): Builder Agent with memory injection and SSE progress
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All acceptance criteria checked
- p95 ≤ 60 s on a real Builder call (manual measurement)
- Memory writes happen only with consent
- Smart-error UI fires on validation failure (manually triggered)
- Status updated, committed, pushed
