# Vibell — Product Capabilities

> **What a user can actually DO in Vibell, where time is saved, what's smart, and which agent powers each step.**
>
> This doc maps user actions → product capability → time saved → agent behind it. Reading top-to-bottom = the user's journey.

---

## Stage 0 — Before signup (the door)

| User does | Capability | Time saved vs. alternative | Agent / system |
|---|---|---|---|
| Lands on `vibell.app` | See live demo strip showing click-to-edit; clear CTA *"Try free"*. | — | Static marketing |
| Joins waitlist | Email captured for early access | — | Static form |
| Signs up | Email or OAuth (Google / GitHub) in <30 s | vs. setting up Vercel + Supabase + Stripe = **2–4 hours** | Supabase Auth |
| Memory consent | Calm one-question prompt: *"Want me to learn your style?"* | — | Memory Agent (waiting) |

---

## Stage 1 — From idea to first preview (target: <5 min)

| User does | Capability | Time saved | Agent |
|---|---|---|---|
| Picks a template OR describes idea | Gallery of 5 universal templates with live previews. *Or* type 1 line → AI matches a template. *Or* paste a URL → Smart Autofill extracts brand. | vs. browsing themes / dribbble for hours = **30–60 min** | **Extractor agent** (Smart Autofill, D-001) |
| Voice-types the description | Mic button → speech-to-text → wizard pre-fills | vs. typing = **2–3 min** | **Voice agent** (D-007) |
| Lets AI fill the wizard | "AI fill the rest" — Coach drafts every field from the user's one-line idea | vs. filling 10 fields manually = **5–10 min** | **Coach** + **Memory** |
| Picks a layout variant | 3 live previews side-by-side, one click | vs. picking from text descriptions = **3 min** + reduces decision regret | Pre-rendered (no AI) |
| "Show me 3 more" | Variant agent generates 3 fresh layouts in parallel | vs. iterating prompt by prompt = **15+ min** | **Variant agent** (3× Haiku 4.5 parallel) |
| Picks a style preset | Minimal / Bold / Playful + fine-tune | vs. picking color combinations from scratch = **20 min** | Pre-defined presets |
| Confirms credit cost | Big violet "Build my app" button. Modal: *"Will use 5 credits. You have 50."* | Transparency = trust, no surprise bills | Credit ledger |
| Watches Builder run | Live progress: caching → generating → validating → saving | vs. silent 60s wait = better perceived speed | **Builder agent** (Sonnet 4.6) + SSE |
| Lands in Workspace | Live preview ready to edit | vs. setting up dev env = **1–3 hours** | **Builder** + **WebContainers** |

**Total time from sign-up to first preview: target ≤ 5 min interactive, ≤ 7 min including Builder generation.**
**Equivalent traditional path: 10–20 hours.** Vibell saves ~95 % of the time.

---

## Stage 2 — Editing (where 90 % of time is spent after the wow moment)

| User does | Capability | Time saved | Agent |
|---|---|---|---|
| Hovers on element → violet outline | Visual confirmation of what's editable | — | Click-to-edit overlay |
| Clicks element → edit panel | Three tabs: **Text** (free), **Style** (free), **AI describe** (2 cr) | vs. opening dev tools / writing CSS = **5–15 min per change** | **UI Editor agent** (Haiku 4.5) |
| Says vague prompt ("make it nicer") | Memory Agent expands prompt with user's known style: *"✦ I'll interpret as: minimal premium, more whitespace, single violet accent. [Edit] [Use as-is]"* | vs. iterating until AI gets it = **3–5 attempts** | **Memory** + **UI Editor** |
| "Show me 3 options" | Variant agent generates 3 alternatives parallel; pick one | vs. asking → judging → re-asking = **5–10 min per change** | **Variant agent** |
| Auto-save | Every 1.5s after a change → snapshot in Versions panel | vs. forgetting to save = **lost work peace** | Background |
| Restore version | Click any past snapshot → instant revert | vs. Git knowledge = **infinite friction for non-devs** | Snapshot system |
| Mobile / Tablet preview | Top-bar icon group switches device frame | vs. opening browser dev tools = **1 min** + better intuition | Device frame component |
| Switch to Studio Mode (paid) | Confirmation modal: 8 cr per layout edit; unlocks Mockup Library | Power users get freedom; cost gates abuse | **Studio agent** (Sonnet 4.6) |
| Add new section in Studio | Pick from extended library or describe what's needed | vs. coding a section from scratch = **2–4 hours** | **Studio agent** |
| Edit JSX in Studio | Code-aware editor for power users | vs. learning React = **weeks** | **Studio agent** + safety rails |
| Build verification (Studio) | After every Studio edit, headless build runs; if broken, auto-rollback with calm message | vs. debugging silently broken site = **hours of frustration** | **Debugger agent** + WebContainers |
| Open Mockup Library (Studio) | Grid of all variants/uploads/explorations for the project; compare, merge, apply | vs. manually screenshotting and comparing = **1+ hour** | Mockup Library + **Variant agent** |
| Upload reference image | Drag/drop screenshot/Figma export → palette + tone extracted, becomes generation seed | vs. describing a reference in words = **10 min and missed nuance** | **Extractor agent** |

---

## Stage 3 — The smart features (what makes Vibell *feel* alive)

These features run in the background or fire intelligently. **The user often doesn't realize they happened — that's the point.**

### Memory of you (Memory agent)
- After 5 uses, knows your aesthetic ("minimal premium with violet accents")
- Knows your phrasings ("spice it up" = "more vibrant color, bolder type")
- Knows your typical project shape (one-pagers vs. multi-page)
- Pre-fills wizard fields with your defaults
- Expands your vague prompts before they reach Builder/UI Editor
- Saves you from repeating yourself

### Continuity Coach
- On every dashboard return, suggests next steps based on project state: *"You haven't added testimonials yet — most landing pages convert better with them. Want me to draft 3 from your existing About copy?"*
- Pickup where you left off
- Never nags; always offers, never imposes

### AI Brand Designer
- One sentence → full brand bible (logo, color system, typography, voice)
- Applies across the project in one click
- Saves 4–8 hours of brand work + designer fees

### AI Debugger (silent)
- Runs in background after every Studio edit, deploy attempt, preview reload
- Reads error → proposes fix → with consent applies
- User-facing free (Vibell absorbs cost)
- The magic: most users will never see an error message

### Smart Autofill from URL
- Paste your existing site / LinkedIn / brand guide PDF
- Agent fetches, screenshots, extracts: logo, palette, typography, voice samples
- Wizard arrives 80 % filled
- 10× faster onboarding for users with existing identity

### Built-in publish analytics
- Every Vibell-published app has tracker pre-installed
- Dashboard shows visitors, top pages, conversions
- AI audit: *"Your hero CTA gets 3 % click — try X for higher conversion"*
- No "set up Google Analytics" step

### Image generation
- Generate hero images, illustrations, product mocks
- Style locked to project palette
- 8 credits per image (user pays — these have real cost)

### Voice mode (free for everyone)
- Talk to Coach in voice
- Voice-edit any element
- Voice-fill the wizard
- Free across all plans (founder direction)

---

## Stage 4 — Publishing (target: <30 s)

| User does | Capability | Time saved | Agent |
|---|---|---|---|
| Clicks Publish | First time: pick subdomain `mysushi.vibell.app`; subsequent: one click | vs. Vercel + DNS + SSL = **2–6 hours** | **Deploy agent** (Vercel API) |
| Watches publish | Live status; toast on done with copy-link button | — | Deploy agent |
| Custom domain (Pro+) | Single input field; we wire DNS + SSL | vs. understanding DNS records = **1–4 hours** | Deploy agent |
| Share | One-click share modal: link + auto-generated OG image | vs. designing OG card = **30 min** | Auto OG render |

---

## Stage 5 — After publish (the loop closes)

| User does | Capability | Time saved | Agent |
|---|---|---|---|
| Sees real visitors | Built-in analytics in workspace top bar | vs. opening Google Analytics = **friction every check** | Analytics |
| Gets AI audit | Periodic suggestion based on real data | vs. guessing at improvements = **most users never iterate** | **Analytics agent** |
| Iterates | Click an audit → applies suggestion via UI Editor | vs. re-design cycle = **days** | UI Editor |
| Shares again | One-click after each meaningful change | — | Share component |

---

## Agent connection map

How agents wire together (called "agent graph"):

```
User intent / action
        │
        ▼
   Orchestrator ─── decides which agent(s) to fire
        │
        ├─► Coach ─────────────── conversational replies, suggestions, onboarding
        │       └─ uses ► Memory Agent (loaded as cached context)
        │
        ├─► Builder ─────────────── wizard payload → siteConfig content
        │       └─ uses ► Memory + Brand Designer (if no brand yet)
        │
        ├─► UI Editor ──────────── single-component edits (text/color/AI)
        │       └─ uses ► Memory (user style + project notes)
        │
        ├─► Variant ─────────────── 3 parallel options
        │       └─ uses ► Memory + UI Editor
        │
        ├─► Studio ──────────────── layout-level changes (paid)
        │       └─ uses ► Memory + Debugger (post-change verify)
        │
        ├─► Brand Designer ──────── full brand bible from a sentence
        │       └─ writes to ► siteConfig.theme + brand_bible table
        │
        ├─► Extractor (Autofill) ── URL → brand assets
        │       └─ feeds into ► wizard payload + Memory
        │
        ├─► Data Schema ─────────── DB design from intent ("save customers")
        │
        ├─► Deploy ──────────────── push to Vercel + subdomain
        │
        ├─► Analytics ───────────── real-data audits + suggestions
        │       └─ uses ► UI Editor for one-click apply
        │
        ├─► Debugger (silent) ──── auto-fix build/runtime errors
        │       └─ runs after every ► Studio + Deploy
        │
        ├─► SEO (deferred to P2)
        ├─► Image Generation
        ├─► Voice ──────────────── speech-to-text → routes to other agents
        ├─► Memory ──────────────── learns user / project (background)
        └─► Continuity Coach ───── proactive next-step suggestions
                └─ uses ► Memory + project state
```

**Key pattern:** Memory feeds nearly every agent (cached system block, ~free). User doesn't repeat themselves; agents share what's known.

---

## Time saved — the bottom line

Versus the traditional path (no-code tool + Stripe + Vercel + DB setup + designer + dev), Vibell delivers a published app in **5–7 min** of interactive time vs. **15–25 hours** of traditional work. That's a **~99 % time saving** at the magic moment, plus ongoing **5–10× speedup** on every edit.

For non-programmers, Vibell isn't 10× faster. **It's the difference between possible and impossible.**
