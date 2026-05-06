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

### Signature accent — Amber Gold
| Token | Hex | Usage |
|---|---|---|
| `--gold-400` | `#FBBF24` | Light/hover state |
| `--gold-500` | `#F59E0B` | **Brand primary.** CTAs, logo, focus rings. |
| `--gold-600` | `#D97706` | Pressed/active state |
| `--gold-700` | `#B45309` | Strong on light bg, accessible text on white |

**Why amber:** semantically tied to *bell* (golden bell), differentiates from purple/blue AI tools, "gold = premium" reinforces positioning, warm = approachable for non-tech audience.

### Semantic
| Token | Hex | Usage |
|---|---|---|
| `--success` | `#10B981` | Saved, deployed, paid |
| `--warning` | `#F59E0B` | (alias of `--gold-500`) approaching credit limit |
| `--danger` | `#EF4444` | Errors, destructive |
| `--info` | `#3B82F6` | Tips, neutral notices |

### Signature gradient
- **Hero gradient (light):** `linear-gradient(135deg, #FFFBEB 0%, #FEF3C7 50%, #FDE68A 100%)`
- **Hero gradient (dark):** `linear-gradient(135deg, #0A0A0A 0%, #1F1408 50%, #3B2510 100%)`
- **Accent gradient (CTAs):** `linear-gradient(135deg, #F59E0B 0%, #D97706 100%)`

### Dark mode
- Background: `#0A0A0A`
- Surface: `#171717`
- Border: `#262626`
- Text: `#FAFAFA`
- Accent retained: `--gold-500`

### Accessibility
- All text/background combos WCAG AA minimum (4.5:1 normal, 3:1 large).
- Gold-500 on white is **not** AA for body text — use `--gold-700` for text on white.
- Always test in dark mode.

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
| Primary | `--gold-500` gradient | none | `--ink-0` | `--gold-600` gradient |
| Secondary | `--paper` | `1px solid --ink-5` | `--ink-1` | `--ink-6` bg |
| Ghost | transparent | none | `--ink-1` | `--ink-6` bg |
| Danger | `--danger` | none | `--paper` | `--danger` darker |

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
PRIMARY: gold #F59E0B
INK:     #0A0A0A → #FAFAFA
RADIUS:  8 / 12 / 999
TYPE:    Geist Sans + Geist Mono
SPACING: 4px base
EASING:  cubic-bezier(0.16, 1, 0.3, 1)
EMOJI:   no
```
