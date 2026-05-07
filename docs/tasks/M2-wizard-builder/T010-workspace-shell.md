# T010 — Workspace shell (3-column editor + Smart/Studio toggle + view switcher)

**Milestone:** M2 — Wizard + Builder
**Status:** Ready
**Estimate:** 4–5 h
**Dependencies:** T004 (app shell), T007 (wizard), T009 (Builder)

---

## Context
After Builder finishes, the user lands in the **Workspace** — a 3-column editor that becomes their daily home. This task ships the shell only: layout, top bar with Smart/Studio toggle and view switcher, sidebar nav, preview placeholder, Coach panel placeholder, Mockup Library panel skeleton. Live preview (WebContainers) and click-to-edit come in M3.

**Reference docs:**
- `/asistenta.md` (READ FIRST)
- `docs/design/workspace.md`
- `docs/design/studio-mode.md`
- `docs/design/mockup-library.md`
- `docs/product/brand.md`

## Goal
A user opens `/app/projects/[id]` after Builder completes and sees a polished, premium workspace shell. They can:
- Inspect the project metadata
- Toggle Smart ↔ Studio (with confirmation modal)
- Toggle view between Full app / Mockups (Studio only)
- Switch device frame (Desktop / Tablet / Phone)
- See "Saved 2s ago" indicator
- Click Publish (stub for now — real flow in M4)
- Open Coach panel
- See Versions list (read-only for this task)

No real preview rendering yet (placeholder); no click-to-edit yet (M3).

## Acceptance Criteria
- [ ] Route: `/app/projects/[id]` (auth-gated)
- [ ] 3-column responsive layout (collapses to stacked on mobile width)
- [ ] **Top bar** with:
  - Wordmark + project name (inline editable)
  - Saved indicator (live from `project_snapshots`)
  - **Smart/Studio toggle** (slide control with violet glow)
  - **View toggle** (Full app / Mockups) — visible only when in Studio Mode
  - Device frame icon group (Desktop / Tablet / Phone)
  - Share button (stub)
  - Publish button (stub — opens "Coming in M4" toast)
- [ ] **Sidebar (left)** with collapsible groups: Pages, Sections, Theme, Data, Versions
- [ ] **Preview pane (center)** placeholder showing "Live preview comes online in M3" with a subtle illustration
- [ ] **Coach panel (right)** stub with chat input + "What can Coach help with?" prompt suggestions
- [ ] **Mockup Library panel** (4th column) — slide-out drawer when in Studio Mode + view = Mockups; shows empty-state ("No mockups yet — generate or upload from Studio")
- [ ] Smart/Studio toggle opens confirmation modal per `docs/design/studio-mode.md`
- [ ] Switching to Studio Mode shows the violet "STUDIO" badge in top bar
- [ ] Visual treatment: dashed violet outlines on hover in Studio (preparatory for M3); solid in Smart
- [ ] Auto-save indicator polls `project_snapshots` updated_at every 5 s
- [ ] Versions sidebar group lists last 50 snapshots with timestamp; click is a no-op for now (M3 will wire restore)
- [ ] Keyboard shortcuts wired (call no-op handlers for now): `⌘K`, `⌘Z`, `⌘⇧Z`, `⌘S`, `⌘P`, `⌘I`, `Esc`
- [ ] Loading state: when Builder is still running (project status='building'), show full-screen `BuildingState` component subscribing to SSE from `/api/agents/builder?projectId=…`
- [ ] Error state: if Builder failed, show smart-error UI with Retry / Back-to-wizard actions
- [ ] All copy in English, no emojis
- [ ] Both light + dark theme polished
- [ ] `pnpm typecheck`, `pnpm build`, `pnpm dev` pass

## Files to create / modify

```
apps/web/src/
├── app/(app)/projects/[id]/
│   ├── page.tsx                                   // server component, loads project + memory consent
│   └── workspace.client.tsx                       // client wrapper for the interactive shell
├── components/workspace/
│   ├── WorkspaceShell.tsx                         // 3-column grid layout
│   ├── TopBar.tsx                                 // wordmark, saved, mode toggle, view toggle, device, share, publish
│   ├── ModeToggle.tsx                             // Smart/Studio slide with confirmation
│   ├── ViewToggle.tsx                             // Full app / Mockups
│   ├── DeviceFrame.tsx                            // Desktop / Tablet / Phone group
│   ├── SaveIndicator.tsx                          // "Saved 2s ago" with poll
│   ├── Sidebar.tsx                                // pages/sections/theme/data/versions
│   ├── PreviewPlaceholder.tsx                     // M3 will replace
│   ├── CoachPanel.tsx                             // chat-input stub + suggestion chips
│   ├── MockupLibraryPanel.tsx                     // empty-state shell
│   ├── BuildingState.tsx                          // SSE-subscribed loader
│   ├── BuilderErrorState.tsx                      // smart-error UI
│   └── shortcuts.ts                               // keyboard shortcut registration
├── lib/workspace/
│   ├── store.ts                                   // Zustand: mode, view, device, sidebarOpen, coachOpen, mockupsOpen
│   └── poll.ts                                    // polling helper for save indicator
└── styles/workspace.css                           // Studio-specific dashed outlines, glow halo
```

## Key UX details

### Top bar layout (left → right)
```
[← back to dashboard]  vibell  /  My Sushi App  · Saved 2s ago         [Smart ⟶ Studio]  [Full app | Mockups]   [Desktop|Tablet|Phone]   Share   Publish
```

### Mode toggle — confirmation modal (entering Studio)
Render the modal exactly as specified in `docs/design/studio-mode.md` § "Entering Studio Mode" (no rewording):

```
Switching to Studio Mode

In Studio Mode, you can change the layout itself — sections, structure,
custom code. More power, more risk.

Each Studio edit costs 8 credits (vs 2 for Smart edits).

Your current snapshot will be saved so you can revert any time.

[ Cancel ]    [ Switch to Studio (free) ]
```

Switching back to Smart Mode is free and instant (no modal).

### View toggle
- Visible only when `mode === 'studio'`.
- Flipping to Mockups slides in the Mockup Library panel from the right (next to Coach), pushes Coach into a tab inside it for now (combined right rail to keep desktop usable).
- Flipping back to Full app retracts Mockups; Coach returns to its rail.

### Studio visual signals
- Top bar gains violet "STUDIO" badge (`bg-violet-600` on light, `bg-violet-600 + glow` on dark).
- Workspace canvas gets a subtle 8px violet vignette around edges.
- All hover outlines on the (M3) preview switch from solid to dashed pattern.

### Coach panel stub
- Heading: *Coach*
- Subhead: *I help when you ask. What's on your mind?*
- 4 chip suggestions: *Make this pop* · *Add a section* · *Explain pricing* · *Help me publish*
- Chat input at bottom (disabled with tooltip "Coach goes live in T0xx" — keep the input visible to test layout)

### Mockup Library panel — empty state (Studio + view=Mockups only)
- Heading: *Mockups*
- Body: *No mockups yet. Switch to Smart Mode and run the wizard to generate your first set, or upload a reference from the top toolbar.*
- "Upload reference" button (stub for M3+)
- "Generate ideas" button (stub)

### Building state (Builder still running)
- Subscribe via EventSource to `/api/agents/builder?projectId=<id>`
- Stages animate per Builder events: started → caching → generating → validating → saving → done
- ETA shown when Builder reports it (typical: 30 s)
- Subtle animation: violet dots in a ring, breathing
- After `done`: smooth transition (250 ms ease-out) to the live workspace shell

### Builder error state
- Renders the structured error from T009
- Calm copy + 2 buttons (Retry · Back to wizard)

## State management

`apps/web/src/lib/workspace/store.ts`:

```ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface WorkspaceState {
  mode: 'smart' | 'studio';
  view: 'full' | 'mockups';
  device: 'desktop' | 'tablet' | 'phone';
  sidebarOpen: boolean;
  coachOpen: boolean;
  enterStudio: () => Promise<void>;
  exitStudio: () => void;
  setView: (v: WorkspaceState['view']) => void;
  setDevice: (d: WorkspaceState['device']) => void;
}

export const useWorkspace = create<WorkspaceState>()(
  persist(
    (set, get) => ({
      mode: 'smart',
      view: 'full',
      device: 'desktop',
      sidebarOpen: true,
      coachOpen: true,
      enterStudio: async () => {
        // confirmation modal handled by component; this is called after confirm
        set({ mode: 'studio' });
        // optional analytics event
      },
      exitStudio: () => set({ mode: 'smart', view: 'full' }),
      setView: (v) => set({ view: v }),
      setDevice: (d) => set({ device: d }),
    }),
    { name: 'vibell-workspace-prefs' }
  )
);
```

## Data fetched on page load (server component)

```ts
// page.tsx (server)
const { data: project } = await supabase
  .from('projects')
  .select('id, name, slug, status, owner_id, template_id, vercel_project_id, live_url, updated_at')
  .eq('id', params.id)
  .single();

if (!project || project.owner_id !== user.id) notFound();

const { data: snapshots } = await supabase
  .from('project_snapshots')
  .select('id, created_at, message')
  .eq('project_id', project.id)
  .order('created_at', { ascending: false })
  .limit(50);

const { data: profile } = await supabase
  .from('user_profiles')
  .select('paused')
  .eq('user_id', user.id)
  .maybeSingle();

return <Workspace project={project} snapshots={snapshots ?? []} memoryConsent={!profile?.paused} />;
```

## Implementation Notes
- Use `framer-motion` for the panel slide animations (Mockups slide-in, Coach reposition).
- Studio dashed outline pattern: Tailwind doesn't ship dashed at desired thickness; use a tiny SVG background or custom CSS class in `styles/workspace.css`.
- Keyboard shortcuts: register in `WorkspaceShell` via `useEffect` + `keydown`; respect input focus (skip when typing in fields).
- The Versions sidebar group lists snapshots from server-fetched data; M3 wires real-time updates.
- Save indicator polls `/api/projects/[id]/last-save` (return only `updated_at` from `projects` and `project_snapshots`) — debounce to 1 req per 5 s.

## Agent Prompt (copy-paste this)

```
You are a senior engineer working on Vibell.

READ FIRST (in order):
- /asistenta.md (founder profile — non-negotiable)
- docs/product/brand.md
- docs/design/workspace.md
- docs/design/studio-mode.md
- docs/design/mockup-library.md
- docs/tasks/M2-wizard-builder/T010-workspace-shell.md (this task)

Task T010: ship the Workspace shell.

Hard rules:
- This task is shell + state + interactions ONLY. No real preview rendering, no click-to-edit (those are M3).
- Strict TypeScript. shadcn/ui + Tailwind only.
- Brand: violet (#7C3AED), near-black, Geist, no emojis.
- Smart/Studio toggle uses the EXACT modal copy from docs/design/studio-mode.md.
- View toggle is visible only in Studio Mode.
- Building state subscribes to /api/agents/builder SSE from T009.
- Both themes must look polished.

Do this:
1. Build all components under components/workspace/.
2. Wire Zustand workspace store with persistence.
3. Build the route page.tsx + workspace.client.tsx pair.
4. Implement device frame switcher (resizes preview placeholder accordingly).
5. Wire keyboard shortcuts as no-op handlers (they'll be wired to real actions in M3).
6. Implement BuildingState (SSE subscriber) + BuilderErrorState.
7. Verify the empty-state Mockup Library panel renders correctly when entering Studio + Mockups view.

When done:
- Update task file Status: Done
- Commit: task(T010): workspace shell with Smart/Studio toggle and view switcher
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All acceptance criteria checked
- Confirmation modal copy matches studio-mode.md exactly
- Both themes render cleanly
- Building state animates SSE stages
- Status updated, committed, pushed
