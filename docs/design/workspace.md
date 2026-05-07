# Workspace — Editor + Preview

**Status:** v0.1
**Why it exists:** This is where 90% of user time is spent after the wizard. It must feel **instant, premium, undoable**.

---

## Layout (3-column desktop, stacked on mobile)

```
┌──────────────────────────────────────────────────────────────────┐
│ Top bar: ← Back  Project name  Saved 2 s ago        Share Publish │
├──────────┬───────────────────────────────────┬───────────────────┤
│ SIDEBAR  │                                    │ COACH             │
│ (240px)  │       PREVIEW (iframe)             │ (320px)           │
│          │                                    │                   │
│ Pages    │  [click any element to edit]       │ Suggestions:      │
│ Sections │                                    │ "Add a testimonial│
│ Theme    │                                    │  section?"        │
│ Data     │                                    │                   │
│ Versions │                                    │ [chat input]      │
│          │                                    │                   │
└──────────┴───────────────────────────────────┴───────────────────┘
```

### Top bar
- Logo wordmark (left, links to dashboard)
- Project name (editable inline)
- Save indicator: "Saved Xs ago" or "Saving…" with a small dot animation
- **Mode toggle:** `Smart ⟶ Studio` slide control (see `docs/design/studio-mode.md`). When in Studio Mode: violet "STUDIO" badge with subtle glow next to the project name.
- Share button (copies project link)
- **Publish** — primary CTA, violet gradient, top-right

### Sidebar (collapsible)
| Section | Purpose |
|---|---|
| Pages | Navigate pages (single-page apps just show "Home") |
| Sections | List of sections in current page (Hero, Features, …) — drag to reorder |
| Theme | Quick global edits: primary color, font, radius |
| Data | If template has DB-backed data, show tables and rows here (CMS-style) |
| Versions | Auto-saved snapshots, click to revert |

### Preview pane (center)
- Live `<iframe>` of the user's app via WebContainers
- 3 view modes (icon toggle in top-center of pane): **Desktop / Tablet / Mobile**
- Hover over any element → violet outline with element name label
- Click → opens edit panel (slides in from right, replacing Coach panel temporarily)

### Coach panel (right)
- Default state: Coach suggestions (4 cards based on project state)
- Chat input at bottom: free-form message to Coach
- Coach replies inline; can include "Apply" buttons for changes

---

## Click-to-edit interaction (the killer feature)

### State machine
```
idle
  ↓ hover element
hovering ─────────────────────────────────┐
  ↓ leave hover                            ▼
idle                                  selected
                                         │
                                  edit panel open
                                         │
                                  ┌──────┼──────┐
                                  ▼      ▼      ▼
                           text       style    ai
                           inline    pickers   prompt
                                  │      │      │
                                  └──────┼──────┘
                                         ▼
                                  apply change
                                         │
                                         ▼
                                  auto-snapshot
```

### Edit panel (right rail, replaces Coach temporarily)
Three tabs:

#### Tab 1: Text
- Inline text editor with Geist Sans rendering
- "Bold / Italic / Link" mini-toolbar
- Character count
- Apply on blur or Enter

**Cost:** 0 credits (no AI involved — just direct text edit).

#### Tab 2: Style
- Color picker (primary, background, text — only properties that make sense for selected element)
- Size: chip selector (sm / md / lg)
- Spacing: slider
- "Reset to default" link

**Cost:** 0 credits (deterministic CSS update).

#### Tab 3: Describe a change (AI)
- Free-text input + voice button
- "Show me 3 options" toggle (default ON)
- Examples below input: clickable suggestions ("Make it bigger", "Add an icon", "More premium feel")
- Submit → UI Editor agent (or Variant agent if "Show me 3 options" is on)

**Cost:**
- Single change: **2 credits**
- 3 variants: **3 credits**

### Variant picker (when 3 options requested)
- 3 cards horizontally, each renders the live result
- Hover → "Pick this" button overlays
- Click → variant becomes active, snapshot is saved
- "Show 3 more" button at bottom — costs another 3 credits
- "Cancel" — closes picker, no charge

---

## Element selection rules

### What's selectable
- Any element with `data-vibe-id` attribute (Builder always tags components)
- Sections (headers, hero, footer)
- Cards within sections
- Individual text/image elements within cards

### What's NOT selectable
- Layout containers without semantic meaning
- Decorative elements (e.g., gradient backgrounds — edit via Theme sidebar)

### Visual feedback
- Hover: 2 px outline in `--violet-500`, label in top-left of element
- Selected: 2 px solid `--violet-600`, slight glow
- Edit panel open: outline becomes pulsating animation

---

## Auto-save

- Debounced: 1.5 s after last change
- Each save = new row in `project_snapshots` (full file map snapshot)
- Top bar shows: "Saving…" → "Saved 2 s ago"
- Versions panel lists last 50 snapshots with timestamp + AI-generated summary ("Changed hero color to violet")
- Click snapshot → preview reverts; "Restore this version" button confirms

**Why full snapshots, not diffs:** simplicity at MVP scale. With ~50 KB per snapshot and 50 snapshots per project, a power user uses ~2.5 MB DB storage. Storage is cheap; compute on diffs is not. Optimize later.

---

## Publish flow

### First time
1. User clicks **Publish** in top bar.
2. Modal: "Pick your URL"
   - Input pre-filled with slug (`mysushi`)
   - Live preview: `mysushi.vibell.app`
   - Availability check (debounced)
3. Click **Publish** → Deploy Agent runs (~10 s)
4. Toast: "Live at mysushi.vibell.app" with copy button
5. Bottom-of-screen card: "Share your work?" with [Add to portfolio] [List on Marketplace]

### Subsequent publishes
- One click. No modal. Toast: "Published — changes are live."
- If URL changes (rare): confirmation modal first.

### Failure
- See `ux-flows.md` G — error states.

---

## Mobile preview toggle

- Icon group in preview top center: `[Desktop] [Tablet] [Phone]`
- Click → preview pane wraps in a phone/tablet frame with realistic dimensions
- Hovering on element still works inside the frame

---

## Data tables (CMS-style)

For templates with DB (Portfolio, Blog, To-do, Booking):

Sidebar **Data** section shows the project's tables:
- Click a table → editable spreadsheet view in main pane (preview moves to right)
- Add row, edit cell inline, delete row
- Schema changes ("add a new field") trigger Data Schema agent

**Cost for schema change:** 3 credits.

---

## Versions panel

- Reverse chronological list, virtualized
- Each entry: timestamp, AI-generated label, "Restore" button
- Click entry → preview enters "preview-only" mode (read-only banner at top)
- Restore → creates a NEW snapshot from the old one (we never delete snapshots)

---

## Keyboard shortcuts (premium feel)

| Shortcut | Action |
|---|---|
| `⌘K` | Command palette (search anything) |
| `⌘Z` / `⌘⇧Z` | Undo / redo |
| `⌘S` | Force save snapshot |
| `⌘P` | Publish |
| `⌘I` | Toggle Coach panel |
| `Esc` | Close any open panel |
| `Tab` while element selected | Cycle to next sibling |

Command palette (`⌘K`) actions:
- Add section…
- Change theme…
- Open versions…
- Talk to Coach…
- Publish

---

## Empty / loading states

| State | UI |
|---|---|
| Workspace opening (Builder running) | Full-screen with Vibell logo + animated dots + ETA + small status messages ("Generating sections…", "Wiring up styles…") |
| Iframe loading | Subtle skeleton inside preview pane |
| Coach loading | Three pulsing dots inside Coach panel |
| Variant generating | Each card shows skeleton until its result lands |
| Publish in progress | Top bar Publish button → "Publishing…" with spinner, becomes "Published 3s ago" then settles back to "Publish" |

---

## Performance targets
- Iframe initial render ≤ 800 ms after Builder finishes
- Click-to-edit panel open ≤ 100 ms
- Single AI edit ≤ 8 s p50, ≤ 15 s p95
- 3-variant generation ≤ 12 s p50 (parallel)
- Auto-save round trip ≤ 300 ms

---

## Open questions
- [ ] Inline text editor in iframe — sandbox rules?  We need cross-origin postMessage to the parent. Confirm WebContainers allow this.
- [ ] Versions retention — keep all forever, or last N? (Suggest: last 50 + monthly checkpoints.)
- [ ] Multi-cursor real-time collab — phase 3 (Team plan).
