# T007 — Wizard UI shell (frontend, 4 steps, no Builder yet)

**Milestone:** M2 — Wizard + Builder
**Status:** Ready
**Estimate:** 4–5 h
**Dependencies:** T004 (app shell, brand tokens)

---

## Context
The Wizard is the heart of the **Guide System** pillar. It walks users through 4 steps and ends by sending a payload to the Builder Agent. This task builds the **frontend shell only** — the UI, state, validation, and a stub submit handler. Builder Agent integration comes in T009.

**Reference docs:**
- `docs/design/wizard.md` (full spec, layout, copy, payload)
- `docs/design/templates.md` (template registry shape)
- `docs/product/brand.md` (visual rules)

## Goal
A user, after sign-up, lands on `/app/new` and walks through 4 steps. State persists across reloads. Final submit posts the payload to a stub endpoint that just logs it and returns a fake `projectId`.

## Acceptance Criteria
- [ ] Route: `/app/new` (inside `(app)` group, requires auth)
- [ ] State stored in Zustand, persisted to `sessionStorage` per `wizardId`
- [ ] Step indicator at top (1–4 with progress bar)
- [ ] All 4 steps implemented per `docs/design/wizard.md`:
  - **Step 1:** Template gallery (4×5 grid using a hardcoded list of 5 templates for now, with placeholder thumbnails) + "Describe your idea" + "Start blank"
  - **Step 2:** Name, 1-sentence, audience chips, feature chips. AI-suggest buttons are stubbed (open a toast "Coming soon — costs 2 credits")
  - **Step 3:** 3 layout variant cards (placeholder images for now). "Show more" stubbed
  - **Step 4:** Style preset cards (Minimal / Bold / Playful) + collapsible fine-tune
- [ ] "Skip" link in top-right of each step (skips remaining steps with defaults)
- [ ] "Back" button navigates to previous step (preserves state)
- [ ] On final submit:
  - POST to `/api/projects/draft` with the wizard payload
  - Endpoint stub validates the payload and returns `{ projectId: 'mock-uuid' }`
  - On 200: redirect to `/app/projects/<id>` (which shows "Project draft saved — Builder integration coming in T009")
- [ ] Voice input button visible but stubbed for MVP (toast "Voice input coming soon")
- [ ] Validation per `docs/design/wizard.md`:
  - Name 2–60 chars, slugifiable
  - Description 5–200 chars
  - At least 1 feature
- [ ] All copy in English, no emojis
- [ ] Looks correct in both themes
- [ ] Animations: 250 ms ease-out between steps
- [ ] Reload mid-wizard → session restored

## Files to create
```
apps/web/src/
├── app/(app)/
│   ├── new/
│   │   └── page.tsx                        // wizard host
│   └── projects/[id]/
│       └── page.tsx                        // stub: "Builder integration coming in T009"
├── app/api/projects/
│   └── draft/route.ts                      // POST stub
├── components/wizard/
│   ├── WizardLayout.tsx                    // shell: header, progress, step area, nav buttons
│   ├── StepIndicator.tsx
│   ├── steps/
│   │   ├── Step1Template.tsx
│   │   ├── Step2About.tsx
│   │   ├── Step3Layout.tsx
│   │   └── Step4Style.tsx
│   ├── TemplateCard.tsx
│   ├── FeatureChip.tsx
│   └── VoiceButton.tsx                    // disabled with tooltip for now
├── lib/wizard/
│   ├── store.ts                            // Zustand store
│   ├── schema.ts                           // zod validators
│   ├── templates.ts                        // hardcoded 5-template registry for now
│   └── slug.ts                             // slug helper
```

## Wizard payload (must match `docs/design/wizard.md`)

```ts
import { z } from 'zod';

export const WizardPayloadSchema = z.object({
  wizardId: z.string().uuid(),
  templateId: z.enum(['landing', 'portfolio', 'blog', 'todo', 'booking', 'blank']),
  name: z.string().min(2).max(60),
  slug: z.string().regex(/^[a-z0-9-]+$/),
  description: z.string().min(5).max(200),
  audience: z.array(z.string()).default([]),
  features: z.array(z.string()).min(1).max(8),
  layout: z.object({
    variantId: z.string(),
    regenerated: z.boolean().default(false),
  }),
  style: z.object({
    preset: z.enum(['minimal', 'bold', 'playful']),
    customColor: z.string().optional(),
    customRadius: z.number().optional(),
    customFont: z.enum(['geist', 'inter', 'space-grotesk']).optional(),
  }),
  voiceUsed: z.boolean().default(false),
  aiAssistanceUsed: z.array(z.string()).default([]),
});

export type WizardPayload = z.infer<typeof WizardPayloadSchema>;
```

## Hardcoded template list for this task (replace with registry in T008+)

```ts
// apps/web/src/lib/wizard/templates.ts
export const TEMPLATES = [
  { id: 'landing', name: 'Landing page', category: 'business', thumb: '/thumbs/landing.png',
    description: 'A single page to launch a product or service.' },
  { id: 'portfolio', name: 'Portfolio', category: 'creative', thumb: '/thumbs/portfolio.png',
    description: 'Showcase your work with project pages.' },
  { id: 'blog', name: 'Blog', category: 'creative', thumb: '/thumbs/blog.png',
    description: 'Write and publish articles with categories.' },
  { id: 'todo', name: 'To-do app', category: 'tools', thumb: '/thumbs/todo.png',
    description: 'A signed-in to-do list with reminders.' },
  { id: 'booking', name: 'Booking', category: 'business', thumb: '/thumbs/booking.png',
    description: 'Let customers book your services online.' },
];

export const FEATURE_CHIPS_BY_TEMPLATE: Record<string, string[]> = {
  landing:   ['pricing', 'testimonials', 'newsletter', 'contact-form', 'faq', 'logos', 'demo-video', 'stats'],
  portfolio: ['about', 'projects', 'skills', 'contact-form', 'resume-download', 'social-links'],
  blog:      ['categories', 'tags', 'subscribe', 'rss', 'comments', 'search'],
  todo:      ['due-dates', 'priorities', 'projects', 'reminders'],
  booking:   ['multi-service', 'payments', 'confirmation-email', 'calendar-sync'],
};

export const AUDIENCE_CHIPS = [
  'small-business', 'freelancer', 'creator', 'student', 'agency',
  'startup', 'non-profit', 'community', 'personal', 'other',
];
```

## Step-by-step UX details (must match `docs/design/wizard.md`)

### Step 1
- Grid of 5 template cards (laid out as 4 cards + 1 in 2nd row for now; placeholder thumbnails are fine)
- "Describe your idea" button below grid → opens a small text input + voice button (voice stubbed)
- "Start blank" small link bottom-right
- Selecting a template advances to Step 2

### Step 2
- Single-column form, generous spacing
- Each field has AI-suggest button (stubbed) and voice button (stubbed)
- "AI fill the rest" button top-right (stubbed)
- Live slug preview under name field (e.g., "URL: <slug>.vibell.app")
- Validation runs on blur and on next-button click

### Step 3
- 3 cards horizontally with placeholder images
- Each card: layout name + 1-line description + click-to-select
- "Show more" button (stubbed)
- For `blank` template, skip this step entirely (advance straight to Step 4)

### Step 4
- 3 large preset cards (Minimal / Bold / Playful)
- "Fine tune" expandable section with color, radius, font controls
- Bottom of page: "This will use 5 credits" note + big violet gradient "Build my app" button
- On click: confirmation modal showing credit cost and current balance, then submits

## Stub submit endpoint

```ts
// apps/web/src/app/api/projects/draft/route.ts
import { NextResponse } from 'next/server';
import { createClient } from '@/lib/supabase/server';
import { WizardPayloadSchema } from '@/lib/wizard/schema';

export async function POST(req: Request) {
  const supabase = createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return NextResponse.json({ error: 'unauthenticated' }, { status: 401 });

  const body = await req.json();
  const parsed = WizardPayloadSchema.safeParse(body);
  if (!parsed.success) return NextResponse.json({ error: parsed.error.format() }, { status: 400 });

  // T009 will: deduct credits, call Builder Agent, persist file map
  // For now: just persist a draft project row and return its id
  const { data, error } = await supabase
    .from('projects')
    .insert({
      owner_id: user.id,
      slug: parsed.data.slug,
      name: parsed.data.name,
      description: parsed.data.description,
      template_id: parsed.data.templateId,
      status: 'draft',
    })
    .select('id')
    .single();

  if (error) return NextResponse.json({ error: error.message }, { status: 500 });
  return NextResponse.json({ projectId: data.id });
}
```

## Implementation Notes
- **Strict TypeScript.** Wizard payload is the contract; it must validate via the zod schema before submit.
- Use **shadcn** components: `Card`, `Button`, `Input`, `Label`, `Badge`, `Dialog`, `Tabs`, `Slider`.
- Slug generation: lowercase, replace whitespace and non-alphanumerics with `-`, dedupe `--`, trim.
- Slug uniqueness check against `projects` table (debounced) — show inline status.
- Animations between steps: `framer-motion` `<AnimatePresence>` with `mode="wait"` and 250 ms slide.
- Persist Zustand state to `sessionStorage` keyed by `wizardId` (uuid generated on first mount).
- Mark stubbed AI buttons with a small "Soon" badge tooltip.

## Agent Prompt (copy-paste this)

```
You are a senior engineer working on Vibell.

Read first (in order):
- docs/product/brand.md
- docs/design/wizard.md (CRITICAL — full spec)
- docs/design/templates.md
- docs/tasks/M2-wizard-builder/T007-wizard-ui.md

Task T007: implement the Wizard UI shell (frontend only, no Builder Agent integration).

Critical rules:
- Use the EXACT copy and validation rules from docs/design/wizard.md.
- Stubs (AI suggest, voice, "Show more", "AI fill") show a "Soon" toast — do not implement Claude calls in this task.
- The submit endpoint persists a draft project row and returns its id. The redirect target is a stub page.
- Brand: violet primary, Geist, no emojis.
- Strict TypeScript. Wizard payload validated by zod before submit.

Do this:
1. Build the WizardLayout shell with step indicator, header, content slot, footer nav.
2. Implement all 4 steps as separate components using the hardcoded template list in lib/wizard/templates.ts.
3. Set up Zustand store persisted to sessionStorage.
4. Add slug helper + uniqueness check (debounced API call to /api/projects/slug-check — also stub).
5. Add the submit endpoint /api/projects/draft that validates payload and inserts a draft row.
6. Add the stub project page that says "Builder integration coming in T009 — your draft is saved".
7. Verify reload mid-wizard restores state.
8. Verify both themes look correct.

When done:
- Update task file Status: Done
- Commit: task(T007): wizard UI shell with 4 steps and stub submit
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All criteria checked
- Manual flow tested: sign in → /app/new → complete 4 steps → submit → land on stub project page
- Reload mid-wizard restores state
- Validation messages calm and brand-aligned
- Status updated, committed, pushed
