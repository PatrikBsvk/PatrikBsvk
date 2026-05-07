# Memory Agent — Personalization that learns the user

**Status:** v0.1
**Why it exists:** Founder direction — *"chci agenta, který se učí chápat, co od něj chci, poznává mě, můj styl vyjadřování, ukládá si o mně poznámky a pomáhá mi posílat lepší prompty"*. This is a competitive moat: base44 and Lovable have no memory of the user.

---

## The promise

> The more you use Vibell, the better it understands you. Without you having to repeat yourself.

After 5 uses, Vibell knows:
- Your aesthetic preference (minimal vs bold, warm vs cool, dense vs airy)
- Your wording habits ("spice it up" might mean "more vibrant" to you specifically)
- Your typical project shape (one-pager landings vs multi-page apps)
- Your common copy length and tone
- Brands you've mentioned as references

After 20 uses, every Builder / UI Editor / Variant call becomes **measurably better** because the memory is automatically attached as context.

---

## Two layers of memory

### Layer 1: Profile (user-level, cross-project)
Long-lived facts about the user that apply to everything they build.

```jsonc
// user_profiles.memory_jsonb
{
  "version": 1,
  "updated_at": "2026-05-06T18:00:00Z",
  "facts": [
    {
      "key": "aesthetic_preference",
      "value": "minimal premium, lots of whitespace, near-black backgrounds with violet accents",
      "confidence": 0.92,
      "evidence": ["picked Minimal style 4/4 times", "said 'cleaner' twice", "removed clutter section once"],
      "first_seen": "2026-05-01",
      "last_reinforced": "2026-05-06"
    },
    {
      "key": "phrasing.spice_it_up",
      "value": "user means 'more vibrant color, slightly bolder type'",
      "confidence": 0.74,
      "evidence": ["said 'spice it up', then accepted variant with brighter accent"],
      "first_seen": "2026-05-03"
    },
    {
      "key": "communication_style",
      "value": "brief, direct, prefers Czech, occasionally uses English tech terms",
      "confidence": 0.88
    },
    {
      "key": "reference_brands",
      "value": ["Stripe", "Vercel", "Notion"],
      "confidence": 0.95
    }
  ]
}
```

### Layer 2: Project Memory (project-scoped)
Facts specific to one project (reset when project is archived).

```jsonc
// project_memories.memory_jsonb
{
  "version": 1,
  "project_id": "uuid",
  "facts": [
    {
      "key": "project.audience",
      "value": "small Czech sushi restaurant, family-owned, traditional values",
      "confidence": 0.99,
      "source": "wizard"
    },
    {
      "key": "project.tone",
      "value": "warm, welcoming, locally rooted (not corporate)",
      "confidence": 0.85
    },
    {
      "key": "project.references",
      "value": ["a local competitor's site (Tabarini)", "user uploaded a screenshot of artisanal layout"],
      "confidence": 0.7
    }
  ]
}
```

---

## How memory is built (the learning loop)

### Sources of signal
| Signal | Layer | Weight |
|---|---|---|
| User text input (wizard, prompts, chat) | Profile + Project | medium |
| Voice transcript | Profile + Project | medium |
| Variant pick (out of 3) | Profile + Project | high (strong preference signal) |
| Explicit edit ("change this color") | Project | medium |
| Section removed entirely | Project | high (negative signal) |
| Section duplicated/extended | Project | high (positive signal) |
| Time spent on a variant before picking | Profile | low |
| Studio Mode action taken | Profile + Project | medium (signals power-user behavior) |

### When the Memory Agent runs
- **Async digest:** every 10 user actions OR every 15 minutes of active session
- **On wizard completion:** reads all wizard inputs, distills 3–5 facts
- **On project archive:** consolidates project facts back into profile if pattern recurring
- **On explicit user feedback:** "I always want X" updates immediately

The Memory Agent **does NOT block** any user-facing action. It runs in the background.

### What the Memory Agent does
- Reads recent actions log (`agent_calls` + `credit_transactions` + user inputs cache)
- Calls Claude (Haiku 4.5) with: existing memory + recent actions → returns proposed updates
- Merges updates into memory:
  - New fact → add with confidence 0.5
  - Reinforced fact → confidence + 0.1, update `last_reinforced`
  - Contradicted fact → confidence - 0.2, possibly drop
- Caps total facts per layer (Profile: 50, Project: 30) — prunes lowest confidence

---

## How OTHER agents use memory

When `runAgent` is called, it injects memory context as a **cached system block**:

```ts
// pseudo
const messages = [
  { role: 'system', content: agentSystemPrompt, cache_control: { type: 'ephemeral' } },
  { role: 'system', content: formatMemory(profileFacts, projectFacts), cache_control: { type: 'ephemeral' } },
  { role: 'user', content: userInput }
];
```

`formatMemory()` produces:

```
# What we know about this user
- Aesthetic: minimal premium, lots of whitespace, near-black with violet accents
- Communication: brief, direct, often switches between Czech and English
- Reference brands: Stripe, Vercel, Notion
- "spice it up" → user means more vibrant color, slightly bolder type

# About this project
- Audience: small Czech sushi restaurant, family-owned
- Tone: warm, welcoming, locally rooted (not corporate)
```

Because this block is cached, **it costs almost nothing per call** (cache_read tokens are 90 % cheaper than fresh input).

---

## Prompt optimization (the "better prompts" part)

When the user types a vague prompt, the Memory Agent can **expand it before sending**.

### Example
User types in click-to-edit panel: *"make this nicer"*

Memory Agent intercepts and produces a richer prompt:

```
User said: "make this nicer"
Based on user profile:
  - prefers minimal premium aesthetic
  - dislikes corporate-looking outputs
Apply this lens:
  Refine the section toward minimal premium — more breathing room, lighter type weight, single accent in violet, remove decorative noise.
```

This expanded prompt goes to the UI Editor / Variant agent. The user **sees** the expansion above the input ("✦ I'll interpret this as: more breathing room, lighter type, single violet accent — based on your style. [Edit] [Use as-is]").

### Two modes
- **Subtle (default):** show a small "✦ understood as: …" hint, user can override
- **Hidden:** when user has opted into "trust me" mode in settings — Memory Agent expansion happens silently

Default is **subtle** because brand voice = transparency.

---

## UI surface (Memory Settings)

`/app/settings/memory` page shows:

1. **Profile facts** — list, sortable by confidence
   - Each row: fact text, confidence bar, evidence count, last reinforced
   - Per-row actions: Edit · Lock (prevent change) · Delete
2. **Per-project memory** — accordion per project
3. **Reset profile** — "Make Vibell forget me" with confirmation modal
4. **Export memory** — JSON download for portability
5. **Pause learning** — toggle

Privacy first: this page is the source of truth, no hidden state.

---

## Storage

```sql
-- Profile (one per user)
create table public.user_profiles (
  user_id    uuid primary key references auth.users on delete cascade,
  memory     jsonb not null default '{"version":1,"facts":[]}'::jsonb,
  paused     boolean not null default false,
  updated_at timestamptz not null default now()
);
alter table public.user_profiles enable row level security;
create policy "user_profiles read own"  on public.user_profiles for select using (auth.uid() = user_id);
create policy "user_profiles write own" on public.user_profiles for all   using (auth.uid() = user_id);

-- Project memories (one per project)
create table public.project_memories (
  project_id uuid primary key references public.projects on delete cascade,
  memory     jsonb not null default '{"version":1,"facts":[]}'::jsonb,
  updated_at timestamptz not null default now()
);
alter table public.project_memories enable row level security;
create policy "project_memories read own" on public.project_memories
  for select using (
    exists (select 1 from public.projects p where p.id = project_id and p.owner_id = auth.uid())
  );
create policy "project_memories write own" on public.project_memories
  for all using (
    exists (select 1 from public.projects p where p.id = project_id and p.owner_id = auth.uid())
  );
```

---

## Onboarding consent (founder-confirmed)

Memory is **opt-in at signup**, not auto-on. The very first interaction after sign-up shows a calm consent prompt:

```
┌──────────────────────────────────────────────────────────┐
│  Want Vibell to learn your style?                        │
│                                                          │
│  When you build with us, we can learn how you            │
│  communicate, what design you like, and what words       │
│  you use. The more you build, the better we understand   │
│  what you want — and we send better prompts to the       │
│  AI on your behalf.                                      │
│                                                          │
│  You can pause, edit, or reset memory any time in        │
│  Settings → Memory.                                      │
│                                                          │
│  [ Not now ]                  [ Yes, learn my style ]    │
└──────────────────────────────────────────────────────────┘
```

- **Default focus on the YES button** — copy framing makes the value obvious.
- **"Not now"** sets `user_profiles.paused = true`. Memory still exists (empty), and a banner in Settings reminds them they can enable any time.
- **Yes** sets `paused = false` and Memory Agent starts observing.

This is consent-based per founder direction (no surprise data collection). GDPR-aligned.

## Privacy & ethics

- **All memory is user-scoped via RLS.** No cross-user leakage.
- **No PII is intentionally extracted.** Memory Agent system prompt explicitly forbids storing names, emails, phone numbers, etc.
- **Deletable.** "Reset memory" wipes both layers.
- **Pausable.** Toggle in settings stops learning.
- **Transparent.** Settings page lists every fact + evidence.
- **GDPR-friendly:** memory export = data portability; reset = right to be forgotten.
- **No selling memory.** It's the user's profile, period.

---

## Cost model

| Operation | Frequency | Model | Cost (USD) | Credits charged to user |
|---|---|---|---|---|
| Background digest | Every 10 actions | Haiku 4.5 | ~$0.005 | **0** (we eat it — it improves their experience) |
| Wizard digest | Per project | Haiku 4.5 | ~$0.01 | 0 |
| Prompt expansion | Per user prompt | Haiku 4.5 | ~$0.003 | 0 |
| Memory load into other agents | Every agent call | (cached) | ~$0.0001 | 0 |

Memory is **free for the user**. We absorb the cost because:
1. It increases retention (users get attached to "their Vibell").
2. It increases output quality, which decreases retry-driven token waste.
3. It's a competitive moat that will be hard to copy.

Estimated cost per active user: **~$0.50/month**. Acceptable.

---

## Open questions
- [ ] Should Profile memory persist across years, or decay confidence over time? (Suggest: decay 5 % per month if no reinforcement.)
- [ ] Multi-language: should Memory Agent reason in user's language or always English internally? (Suggest: English internal, surface in user's language.)
- [ ] Should we offer a "memory marketplace" where users share style profiles? (Phase 3+, interesting but privacy minefield.)
- [ ] Should other users' Coach interactions (anonymized) feed a global Vibell model that learns common patterns? (Phase 3, strict opt-in.)

---

## Implementation tasks (will be queued)
- T0xx — `user_profiles` and `project_memories` tables (small migration after T003)
- T0xx — Memory Agent (`packages/agents/src/agents/memory.ts`)
- T0xx — Memory loading helper used by `runAgent`
- T0xx — Settings page (`/app/settings/memory`)
- T0xx — Prompt expansion UI in click-to-edit panel
