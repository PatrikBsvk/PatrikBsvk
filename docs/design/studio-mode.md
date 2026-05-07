# Studio Mode — Paid Layout Freedom

**Status:** v0.1
**Why it exists:** Founder direction (2026-05-06): "layout locked dává smysl, ale v případě že uživatel chce změnit, dej mu prostor, za který si zaplatí". This doc defines that escape hatch.

---

## The two modes (mental model)

| | **Smart Mode** (default) | **Studio Mode** (paid) |
|---|---|---|
| Layout | Locked. AI fills content within fixed sections. | Unlocked. AI can add/remove/reorder sections, edit JSX. |
| Risk of broken output | Very low | Higher (AI can break things) |
| Token cost | Cheap (small prompts, cached templates) | Expensive (full file context, no caching benefit) |
| Speed per edit | 5–8 s | 15–30 s |
| Who uses it | 95 % of users, 99 % of edits | Power users, complex one-offs |
| Plan availability | All plans | Free + Basic: limited trial. Standard+: unlimited. |

**Default is always Smart Mode.** Studio Mode is opt-in per project, with a clear "you are leaving the safe zone" moment.

---

## What Studio Mode unlocks

Things you CANNOT do in Smart Mode that Studio Mode allows:

1. **Add a section type that's not in the template's whitelist** (e.g., add a 3D model viewer to a Landing page).
2. **Reorder or remove core sections** (e.g., remove the Footer entirely).
3. **Modify layout JSX** (change two-column to three-column, change grid to flex, etc.).
4. **Restructure the page hierarchy** (split into multiple pages, add navigation between them).
5. **Override Tailwind classes with custom styles** (custom CSS in a scoped sheet).
6. **Import an external library** (with safety review for security).
7. **Open the Mockup Library** — the visual gallery of all variants, references, and explorations for this project. See `docs/design/mockup-library.md`. Studio Mode is the home of the Mockup Library; Smart Mode hides it.

---

## How the UX feels

### Entering Studio Mode
- A toggle in the workspace top bar: `Smart  ⟶  Studio` (slide control)
- Click → confirmation modal:

```
┌────────────────────────────────────────────┐
│  Switching to Studio Mode                  │
│                                            │
│  In Studio Mode, you can change the        │
│  layout itself — sections, structure,      │
│  custom code. More power, more risk.       │
│                                            │
│  Each Studio edit costs 8 credits          │
│  (vs 2 for Smart edits).                   │
│                                            │
│  Your current snapshot will be saved so    │
│  you can revert any time.                  │
│                                            │
│  [ Cancel ]    [ Switch to Studio (free) ] │
└────────────────────────────────────────────┘
```

- Switching back to Smart Mode is always free and instant.

### Inside Studio Mode
- Workspace top bar gets a **violet "STUDIO" badge** with subtle glow.
- Edit panel (right rail) gets a 4th tab: **"Layout"** with options:
  - Add section
  - Remove section
  - Move section up/down
  - Edit JSX (advanced — opens a code-aware editor with the file)
- Coach tone shifts: more cautious — "This is a structural change. Want me to make a snapshot first?"
- Auto-save snapshots are tagged `studio: true` for telemetry.

### View toggle — Mockup vs Full app
Studio Mode adds a **view-mode switcher** in the workspace top bar (founder direction): the user can flip between two ways of working without losing context.

| View | What it shows | Best for |
|---|---|---|
| **Full app** (default) | The complete assembled site/app live in the iframe | Working on the real project, click-to-edit, publish |
| **Mockups** | The Mockup Library grid takes center stage; preview moves to a side rail | Ideating, comparing, iterating on visuals before applying to project |

Switching between views is **instant and preserves all panels' state**. Coach panel and Memory Agent observe both views. The Mockup Library exists only in Studio Mode (Smart Mode hides it entirely).

### Visual differentiation
- The whole canvas gets a subtle violet vignette around the edges (peripheral signal that you are in advanced mode).
- Element outlines on hover are still violet, but with a dashed pattern (vs solid in Smart Mode) — communicates "different rules apply here".

---

## Pricing & plan gating

### Per-action credit cost
| Action | Smart | Studio |
|---|---|---|
| Text edit | 0 | 0 |
| Style edit (color, spacing) | 0 | 0 |
| AI content edit | 2 | 2 (no change — same agent) |
| AI variants (3) | 3 | 3 |
| **Add section** | n/a | **8** |
| **Remove section** | n/a | **5** |
| **Reorder sections** | n/a | **3** |
| **Edit layout JSX** (per request) | n/a | **8–15** (depends on scope) |
| **Custom CSS** (per request) | n/a | **5** |
| **Add custom page** | n/a | **15** |

Why these numbers: each Studio action sends much more context to Claude (full layout files, related components) and outputs full JSX, so cost is roughly 4–6× a Smart edit. We charge 4–8× to keep margin.

### Plan gating
| Plan | Studio access |
|---|---|
| Free | **First 3 Studio edits free** (so users feel the power before paying), then locked. Upgrade prompt. |
| Basic | First **10 Studio edits/month** included; rest charged at standard credit cost. |
| Standard | Unlimited Studio edits at standard credit cost. |
| Pro | Unlimited Studio edits + **20% credit discount** on Studio actions only. |
| Team | Pro benefits + collaborative Studio (multiple cursors). Phase 3. |

This creates a clean upgrade ladder: Free users *feel* Studio, get hooked, upgrade to keep using it.

---

## Safety rails (so power doesn't break the user's app)

1. **Auto-snapshot before every Studio action.** Always.
2. **Build verification.** After every Studio edit, run `pnpm build` headlessly in the WebContainer. If it fails, the change is rolled back automatically with a calm message: *"That change broke the build. We rolled it back."*
3. **JSX schema check.** Studio edits must result in valid TSX that imports only whitelisted packages.
4. **Layout drift detection.** If Studio edits remove the `data-vibe-id` tags, click-to-edit stops working on that section. We warn the user before applying.
5. **One-click "back to Smart".** A user can always revert to the last Smart-Mode snapshot, dropping all Studio changes.

---

## What Studio Mode is NOT

- **Not** a code editor for power users. There's no Cursor / Bolt experience inside Vibell.
- **Not** an unrestricted prompt box. Studio still uses agents with constrained outputs.
- **Not** required for most users. We measure success by how few users *need* it (target: < 10 % of weekly active users use Studio Mode in a typical week).

---

## Telemetry to watch

- `studio_mode_entries` per user per week
- `studio_action_breakdown` (add section vs JSX edit vs …)
- `studio_rollbacks` (build failed → rolled back) — high rollback rate = improve agent
- `studio_to_paid_conversion` — Free users hitting the 3-edit limit, did they upgrade?

---

## Implementation plan (where it lives)

- New agent: **Studio Agent** (Sonnet 4.6, full template context, larger max_tokens, no caching benefit)
  - Lives in `packages/agents/src/agents/studio.ts`
- Workspace UI: Studio Mode toggle, badge, "Layout" tab (M3+ task — see T0xx, to be drafted)
- `agent_calls.studio_mode` boolean column for telemetry
- Build-verification subprocess in WebContainer (M3 follow-up task)

---

## Open questions
- [ ] After how many Studio edits should we proactively suggest "you might be happier exporting to GitHub and editing in your IDE" (Pro plan feature)?
- [ ] Should Studio Mode be **per-project** (you set it once per project) or **per-action** (toggle for one action)? Current design: per-project session, so you're either in or out. Simpler mental model.
- [ ] Should Studio Mode unlock the ability to chat with the **Orchestrator** directly (multi-step plans)? Probably yes, in M5+.
