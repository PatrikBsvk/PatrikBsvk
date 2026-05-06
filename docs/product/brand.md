# Vibell — Brand Bible

**Status:** v0.1 (working draft, founder-approved direction)
**Last updated:** 2026-05-06

---

## 1. Brand essence

| | |
|---|---|
| **Name** | Vibell |
| **Domain** | vibell.app |
| **Three words** | modern · professional · premium |
| **Tone references** | Stripe (clarity), Vercel (minimal), Notion (interactive warmth) |
| **NOT** | playful · childish · cluttered · loud |

## 2. One-liner

- **EN:** *"Have an idea. Vibell builds it."*
- **CZ:** *"Měj nápad. Vibell ti ho postaví."*

## 3. Brand promise

> **You ring the bell, we build the app.**
>
> Vibell turns ideas into published web apps without code, terminals, or DevOps. Premium tooling, designed so anyone can use it.

## 4. Voice & tone

### Voice (always)
- **Clear over clever.** No idioms in product copy.
- **Confident, not boastful.** "Your app is live." not "🎉 Woohoo! Look what you made!"
- **Warm, not familiar.** "Welcome back, Patrik" — not "Hey buddy!"
- **Specific over generic.** "Saved 2 seconds ago" — not "Saved recently."

### Tone modulation (by context)
| Context | Tone |
|---|---|
| Onboarding | Encouraging, simple, short |
| Editing workspace | Quiet, helpful, only speaks when needed |
| Errors | Calm, solution-oriented, no jargon |
| Marketing site | Confident, aspirational, factual |
| Billing | Direct, transparent, no surprises |

### Microcopy do/don't

| ❌ Don't | ✅ Do |
|---|---|
| "Oops! Something went wrong 😅" | "Couldn't save. Retry, or check your connection." |
| "Awesome! You're a rockstar 🎸" | "Your app is live at sushi.vibell.app." |
| "Loading data..." | "Loading projects." |
| "Oh no, the AI got confused" | "We couldn't generate that section. Try rephrasing or pick a template." |
| "Click here to deploy" | "Publish" |

### Forbidden words
- *awesome*, *rockstar*, *ninja*, *guru*
- *democratize*, *revolutionize*, *disrupt*
- *AI-powered* (we say *built with AI* or just don't mention)
- Excessive emojis. **Default: no emojis in product UI.**

## 5. Color palette

### Foundation (neutrals)
| Token | Hex | Usage |
|---|---|---|
| `--ink-0` | `#0A0A0A` | True near-black. Headings, primary text. |
| `--ink-1` | `#171717` | Body text on white. |
| `--ink-2` | `#404040` | Secondary text. |
| `--ink-3` | `#737373` | Tertiary, captions. |
| `--ink-4` | `#A3A3A3` | Disabled, placeholders. |
| `--ink-5` | `#D4D4D4` | Borders, dividers. |
| `--ink-6` | `#F5F5F5` | Surface (cards, panels). |
| `--ink-7` | `#FAFAFA` | Background. |
| `--paper` | `#FFFFFF` | Pure white surface. |

### Signature accent — Royal Violet
| Token | Hex | Usage |
|---|---|---|
| `--violet-400` | `#A78BFA` | Light/hover state, glow |
| `--violet-500` | `#8B5CF6` | Hover, secondary accent |
| `--violet-600` | `#7C3AED` | **Brand primary.** CTAs, logo, focus rings. |
| `--violet-700` | `#6D28D9` | Pressed/active state |
| `--violet-800` | `#5B21B6` | Strong on light bg, accessible text on white |
| `--violet-950` | `#2E1065` | Deep tone, dark-mode panels |

**Why violet + black:** founder direction. Premium, modern, tech-forward (Linear, Anthropic, Stripe-grade signature). Differentiates from base44 (lighter, friendlier) by leaning serious/luxurious. Violet on near-black communicates "premium AI tool" instantly.

### Semantic
| Token | Hex | Usage |
|---|---|---|
| `--success` | `#10B981` | Saved, deployed, paid |
| `--warning` | `#F59E0B` | Approaching credit limit |
| `--danger` | `#EF4444` | Errors, destructive |
| `--info` | `#3B82F6` | Tips, neutral notices |

### Signature gradients
- **Hero gradient (light):** `linear-gradient(135deg, #FAFAFA 0%, #F3EEFE 50%, #E9D5FF 100%)` — subtle violet wash on light backgrounds
- **Hero gradient (dark):** `linear-gradient(135deg, #0A0A0A 0%, #1B0F3F 60%, #2E1065 100%)` — deep black to violet
- **Accent gradient (CTAs):** `linear-gradient(135deg, #8B5CF6 0%, #7C3AED 50%, #6D28D9 100%)`
- **Premium glow:** `box-shadow: 0 0 40px rgba(124, 58, 237, 0.35)` — used sparingly on hero CTA, brand mark

### Dark mode (the hero look)
Dark mode is **first-class**, not an afterthought. The premium feel of Vibell lives in the dark theme; the marketing site hero defaults to dark.

- Background: `#0A0A0A` (true near-black)
- Surface: `#0F0F12` (slightly elevated)
- Surface raised: `#17141F` (cards on dark)
- Border: `#262626`
- Border violet-tinted: `rgba(139, 92, 246, 0.15)` (subtle purple line on cards)
- Text primary: `#FAFAFA`
- Text secondary: `#A3A3A3`
- Accent retained: `--violet-600`
- Glow halo on focused/active CTAs: `--violet-500` with 30% opacity

### Accessibility
- All text/background combos WCAG AA minimum (4.5:1 normal, 3:1 large).
- `--violet-600` on white is borderline AA for body — use `--violet-700` for text on white.
- `--violet-400` is the readable choice on `#0A0A0A`.
- Always test in both light and dark.

## 6. Typography

### Type families
| Family | Usage | Source |
|---|---|---|
| **Geist Sans** | Display, headings, body, UI | `geist-font` package, free |
| **Geist Mono** | Code, slugs, IDs | `geist-font` package |

Geist is Vercel's open-source geometric sans. Premium feel, excellent legibility, free to use commercially.

### Scale (rem-based, 16px root)
| Token | Size | Line height | Weight | Usage |
|---|---|---|---|---|
| `text-display` | 3.75rem (60px) | 1.05 | 600 | Hero headline |
| `text-h1` | 2.25rem (36px) | 1.15 | 600 | Page title |
| `text-h2` | 1.5rem (24px) | 1.25 | 600 | Section title |
| `text-h3` | 1.25rem (20px) | 1.3 | 600 | Subsection |
| `text-body-lg` | 1.125rem (18px) | 1.6 | 400 | Lead paragraph |
| `text-body` | 1rem (16px) | 1.6 | 400 | Default body |
| `text-body-sm` | 0.875rem (14px) | 1.5 | 400 | Secondary, labels |
| `text-caption` | 0.75rem (12px) | 1.4 | 500 | Captions, badges |

### Rules
- **Never** use weight 400 for headings — looks weak at large sizes. Always 500 or 600.
- **Never** use weight 700 — Geist 600 already reads strong; 700 looks heavy.
- Body text always Geist Sans 400 with line-height 1.6.
- Letter spacing: `-0.02em` for display sizes only, default elsewhere.

## 7. Logo

### Wordmark (primary)
- Lowercase: **vibell**
- Set in Geist Sans 600
- Letter spacing: `-0.04em`
- Color: `--ink-0` on light, `--paper` on dark

### Symbol (optional, for app icon and small contexts)
- Concept: stylized bell shape that doubles as a sound/vibration wave
- 1:1 square mark
- Single color (gold) or two-tone (gold + ink)

> **Status:** symbol design is TBD. For MVP, wordmark only is acceptable. Symbol can be added in M2 polish phase.

### Usage rules
- **Clear space:** minimum padding around logo = height of "v" character.
- **Minimum size:** 80px width for digital, 20mm for print.
- **Don't:** stretch, recolor outside palette, add effects, place on busy backgrounds.

## 8. Iconography

- **Library:** Lucide Icons (already used by shadcn/ui).
- **Stroke width:** 1.5px (slightly lighter than default 2px for premium feel).
- **Size:** 16, 20, 24px standards.
- **Color:** inherits from text color.
- **No** custom icons in MVP — Lucide is enough.

## 9. Components language

### Shape
- **Border radius:** 8px (small elements), 12px (cards, modals), 999px (pills, avatars).
- **Cards:** 1px border `--ink-5`, no shadow by default. Subtle shadow on hover (`0 4px 12px rgba(0,0,0,0.04)`).
- **Buttons:** 8px radius, no shadow. Primary = gold gradient. Secondary = white with border.
- **Modals:** 12px radius, white surface, `0 16px 48px rgba(0,0,0,0.12)` shadow.

### Spacing
- 4px base unit. Scale: `4, 8, 12, 16, 24, 32, 48, 64, 96`.
- Generous whitespace. Default page max-width 1280px, content max-width 720px.

### Motion
- **Premium ≠ static.** Subtle motion is fine, never bouncy.
- Easing: `cubic-bezier(0.16, 1, 0.3, 1)` (smooth ease-out).
- Durations: 150ms (micro-interactions), 250ms (transitions), 400ms (page transitions).
- Reduce motion: respect `prefers-reduced-motion`.

### Buttons (primary spec)
| Variant | Background | Border | Text | Hover |
|---|---|---|---|---|
| Primary (light) | violet gradient | none | `--paper` | shift to violet-700 |
| Primary (dark) | violet gradient + glow | none | `--paper` | brighter glow |
| Secondary | `--paper` (or `--ink-1` dark) | `1px solid --ink-5` | `--ink-1` | `--ink-6` bg |
| Ghost | transparent | none | `--ink-1` | `--ink-6` bg |
| Danger | `--danger` | none | `--paper` | `--danger` darker |

Primary CTA always uses the violet gradient (`#8B5CF6 → #6D28D9`). On dark mode it has a soft glow (`box-shadow: 0 0 24px rgba(124,58,237,0.4)`). This is the single most recognizable Vibell visual — keep it consistent.

## 10. Imagery & illustrations

- **Photos:** rare. When used, neutral, premium, minimal subjects (objects, hands, devices). No stock people.
- **Illustrations:** geometric, minimal, monochrome with single gold accent. Custom-made, never stock.
- **Mockups:** browser frames in flat outline style (no realistic Mac chrome).
- **No mascot** (founder decision).

## 11. Naming conventions (product features)

| Internal name | User-facing label (EN) | User-facing (CZ) |
|---|---|---|
| Wizard | Builder | Stavitel |
| Workspace | Editor | Editor |
| Project | App | Aplikace |
| Snapshot | Version | Verze |
| Marketplace | Vibell Store | Vibell Store |
| Coach | (no separate name; just "Help") | Nápověda |

## 12. Open decisions (still TBD)

- [ ] **Symbol/mark design** — wordmark-only for MVP, symbol added in M2
- [ ] **App icon** — derived from symbol, designed in M2
- [ ] **Marketing site illustrations** — outsource or use minimal CSS-only?
- [ ] **Sound design** — does Vibell make a *bell* sound on publish? (delightful but optional)

## 13. Quick reference card

```
PRIMARY:   violet  #7C3AED   (CTAs, focus, brand)
GLOW:      violet  rgba(124,58,237,0.35)  (hero CTA on dark)
INK:       #0A0A0A → #FAFAFA
GRADIENT:  #8B5CF6 → #6D28D9  (primary CTA)
HERO BG:   #0A0A0A → #1B0F3F → #2E1065  (dark hero)
RADIUS:    8 / 12 / 999
TYPE:      Geist Sans + Geist Mono
SPACING:   4px base
EASING:    cubic-bezier(0.16, 1, 0.3, 1)
DARK:      first-class, not afterthought
EMOJI:     no
```
