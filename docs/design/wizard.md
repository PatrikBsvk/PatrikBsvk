# Wizard — The 4-Step Builder

**Status:** v0.1
**Why it exists:** The Wizard is the **Guide System pillar**. It's the difference between Vibell and "blank prompt" tools (Lovable, Bolt, base44).

---

## North-star metric for the wizard
**Wizard completion rate ≥ 80 %** for first-time users (industry baseline for guided onboarding is 50–60 %; we aim higher because each step previews progress).

## Hard rules
1. **Total interactive time ≤ 90 seconds.**
2. **Zero mandatory free-text fields.** Every input has an AI-fillable default.
3. **Every step previews the next.** User always sees what's coming, never feels lost.
4. **Skip-anywhere.** Top-right "Skip" → AI fills remaining steps with sensible defaults.
5. **Voice-first option.** A microphone icon at every text input lets the user speak.

---

## Step 1 — Pick a starting point

### Layout
- Header: "What do you want to build?"
- Subheader: "Pick a template, or describe your idea and we'll find one."
- 4×5 grid of template cards (20 templates)
  - Each card: thumbnail (animated GIF showing the template scrolling), name, 1-line description
  - Hover: live preview opens in modal
- Above grid: search input + "Describe your idea" button (opens free-form text + voice input)
- Bottom right: "Start blank" link (small, intentionally de-emphasized)

### Template categories (chips above grid)
- All · Business · Creative · Personal · Marketplace · Tools · Education

### What we record
```ts
{
  templateId: string | 'blank',
  arrivedVia: 'gallery' | 'description' | 'blank' | 'search',
  rawDescription?: string  // if user used "Describe your idea"
}
```

### Description-to-template fallback
If user picks "Describe your idea":
1. Show 1 input + voice button. Placeholder: *"A landing page for my coffee shop"*
2. Submit → Coach agent (Haiku 4.5) returns a ranked list of 3 matching templates
3. Show those 3 as large cards. User picks one or "None of these → Start blank"

This costs **2 credits** (voice → text + Coach matching). Free for first wizard.

---

## Step 2 — Tell us about it

### Layout
Single-column form, large inputs, generous whitespace. Each field has:
- Label (Geist 600, ink-1)
- AI-suggest button on the right of the input ("✨ Suggest")
- Voice button next to AI-suggest

### Fields
| Field | Type | Required | AI fills with |
|---|---|---|---|
| Name | Text input | Yes | Capitalized template noun ("My Landing Page") |
| One sentence | Text area (1 row) | Yes | Template's default tagline |
| Audience | Multi-chip selector | No | Most common audience for that template |
| Features | Multi-chip selector (predefined per template + "Add custom") | Pick at least 1 | 3 most-popular features for that template |

### Predefined feature chips per template (example: Landing)
- Pricing tiers
- Testimonials
- Newsletter signup
- Contact form
- FAQ
- Logo cloud
- Demo video
- Stats counter

### "AI fill the rest" button (top-right)
Triggers Coach agent → fills all remaining fields with sensible defaults from the template + name + sentence. **Costs 2 credits.** First time free.

### What we record
```ts
{
  name: string,
  description: string,         // the 1-sentence
  audience: string[],
  features: string[],
  voiceUsed: boolean,
  aiFilled: boolean
}
```

### Validation
- Name: 2–60 chars, slugifiable
- Description: 5–200 chars
- Features: at least 1, max 8

### Inline validation tone
Per brand voice — calm and specific:
- "Make this a bit longer (5+ characters)." — not "Field too short!"

---

## Step 3 — Mockup preview

### Layout
- 3 cards side-by-side, each showing a different layout variant
- Each card is a **real rendered preview** (not a static image), at 0.6× scale
- Below each card: 1-line description of the variant
- Click a card → it expands to a larger preview, "Use this layout" button appears

### Variants per template (Landing example)
1. **Centered hero** — title + sub centered, CTA below, sections stack
2. **Split hero with image** — text left, hero image / illustration right
3. **Gradient background** — full-bleed gradient hero, sections in cards

### How variants are generated
- Pre-rendered at template build time (in `packages/templates/<id>/variants/<n>.tsx`)
- Filled with the user's name + description + features → live HTML
- **NO Claude call here.** Cost: 0 credits.

### "Show more" button
Below the 3 cards. Click → generates 3 additional variants via Variant Agent (Haiku ×3 parallel). **Costs 3 credits.**

### What we record
```ts
{
  layoutVariantId: string,
  showMoreClicks: number,
}
```

---

## Step 4 — Style

### Layout
- Three large preview cards (each shows the chosen layout variant from Step 3 with that style applied)
- Below: a "fine tune" expandable panel with:
  - Primary color (color picker, default = brand violet `#7C3AED`)
  - Border radius (slider)
  - Font (3 options: Geist / Inter / Space Grotesk)

### Style presets
1. **Minimal** — heavy whitespace, neutral palette, sans serif, soft borders
2. **Bold** — high contrast, strong type, gradients, generous shapes
3. **Playful** — rounded corners, more colors allowed, friendlier voice in copy

### CTA at bottom
Big violet gradient button: **"Build my app"**

When clicked:
- Confirmation modal: "This will use 5 credits. You have 50."
- "Build" → wizard ends, Builder Agent runs.

### What we record
```ts
{
  stylePreset: 'minimal' | 'bold' | 'playful',
  customColor?: string,      // if user changed
  customRadius?: number,
  customFont?: string,
}
```

---

## Final wizard payload (sent to Builder)

```json
{
  "wizardId": "uuid",
  "userId": "uuid",
  "templateId": "landing-saas",
  "name": "Sushi Master",
  "slug": "sushi-master",
  "description": "An online ordering page for the best sushi in town.",
  "audience": ["foodies", "small-business"],
  "features": ["pricing", "testimonials", "contact-form"],
  "layout": {
    "variantId": "centered",
    "regenerated": false
  },
  "style": {
    "preset": "minimal",
    "customColor": "#7C3AED"
  },
  "voiceUsed": false,
  "aiAssistanceUsed": ["step2-fill"]
}
```

This is the *exact* payload `runAgent({ agent: 'builder', input })` receives.

---

## State management

- Wizard state lives in **Zustand** store (client-side) until Step 4 confirm.
- On Step 2/3/4 transition, we **save a draft** server-side (`projects` row with `status='draft', wizard_state jsonb`).
- If user reloads, dashboard shows "Continue setup" card.

---

## Empty states

- **No matching template** in description-to-template: Coach offers "We didn't find a match. Want to start blank or try a different description?"
- **0 credits** when clicking Build: Upgrade modal (see ux-flows.md G).

---

## Acceptance criteria for wizard implementation
- All 4 steps render in both light + dark theme matching brand.md
- Voice input works (Web Speech API, Chrome / Safari / Edge supported; Firefox: graceful fallback to text-only)
- Skip button at every step works
- AI fill button works and is gated by credit balance
- Wizard state persists across page reloads
- Final payload is correctly structured
- Animations: 250 ms ease-out between steps, no jarring jumps
- Lighthouse a11y score ≥ 95 on each step

---

## Open questions
- [ ] Should we offer a **Pro shortcut**: "Upload a Figma link / screenshot, AI matches it"? (Phase 2.)
- [ ] Voice language: EN only at launch, CZ phase 2?
- [ ] Wizard for cloning a marketplace app: skip Step 1, jump straight to Step 2 with cloned values pre-filled?
