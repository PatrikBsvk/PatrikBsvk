# Templates — Starter Scaffolds

**Status:** v0.1
**Why this matters:** Templates are the **rails** that make Vibell ship reliable apps. Builder fills content, but doesn't restructure layouts. Less surface for AI to mess up = higher quality + fewer tokens.

---

## Template registry — MVP set (5 templates)

| ID | Name | DB needed | Variants | First built |
|---|---|---|---|---|
| `landing` | Landing page | No | 3 | **Yes (M2)** |
| `portfolio` | Portfolio | Yes (small) | 3 | M2 |
| `blog` | Blog | Yes | 3 | M3 |
| `todo` | To-do app | Yes | 2 | M3 |
| `booking` | Booking | Yes | 2 | M3 |

After MVP, we expand to 20+ templates and let creators publish their own.

---

## Template manifest format

Every template lives in `packages/templates/<id>/` and exposes:

```ts
// packages/templates/<id>/manifest.ts
import { TemplateManifest } from '@vibell/templates/types';

export const manifest: TemplateManifest = {
  id: 'landing',
  displayName: 'Landing page',
  description: 'A single-page site to launch a product, service, or idea.',
  category: 'business',
  thumbnail: '/thumbs/landing.png',
  previewUrl: '/previews/landing',
  estimatedBuildSeconds: 30,
  variants: ['centered', 'split', 'gradient-bg'],
  sections: [
    { id: 'hero', required: true,  configurable: ['title', 'subtitle', 'ctaPrimary', 'ctaSecondary'] },
    { id: 'logos', required: false, configurable: ['logos'] },
    { id: 'features', required: false, configurable: ['items'] },
    { id: 'how-it-works', required: false, configurable: ['steps'] },
    { id: 'pricing', required: false, configurable: ['tiers'] },
    { id: 'faq', required: false, configurable: ['questions'] },
    { id: 'cta', required: true,  configurable: ['title', 'cta'] },
    { id: 'footer', required: true, configurable: ['links'] },
  ],
  features: ['pricing', 'testimonials', 'logos', 'newsletter', 'contact-form', 'faq', 'demo-video'],
  databaseSchema: [],          // SQL for any tables (empty for static landing)
  scaffold: (input) => ({...}) // returns FileMap given wizard payload
};
```

The `scaffold` function returns a file map keyed by path — the Builder Agent then refines content within these files.

---

## Template 1 — Landing page (the first one we build)

### Goal
A single-page site that converts visitors. Works for SaaS, agencies, shops, products, courses, anything where someone needs **one page that explains a thing and gets a click**.

### Sections (in order)
1. **Hero**
   - Title (`<h1>`, display size)
   - Subtitle (1–2 sentences)
   - Primary CTA (button)
   - Secondary CTA (link, optional)
   - Optional: hero image / illustration / gradient background

2. **Logo cloud** (optional)
   - "Trusted by" prefix
   - 4–8 logos in a grayscale row

3. **Features**
   - Section title
   - 3 cards (icon, title, body)

4. **How it works** (optional)
   - Section title
   - 3 numbered steps

5. **Pricing** (optional)
   - Section title
   - 1, 2, or 3 tier cards
   - "Most popular" highlight on middle tier

6. **Testimonials** (optional)
   - Section title
   - 1 hero quote OR 3-card grid

7. **FAQ** (optional)
   - Section title
   - 4–6 expandable Q&A

8. **CTA section**
   - Title + button (last chance)

9. **Footer**
   - Logo + tagline
   - Link columns
   - Social links (optional)
   - Copyright

### Variants
- **Centered hero** (`centered`) — title + sub + CTA centered, page is single column. Default.
- **Split hero** (`split`) — text left, image/illustration right. Two-column hero, single-column rest.
- **Gradient hero** (`gradient-bg`) — full-bleed gradient background on hero, rest of page on white.

### Configurable theme
- Primary color (default: violet `#7C3AED`)
- Border radius (default: 12 px)
- Font (default: Geist)

### File map (scaffold output, before Builder fills content)

```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx                     // composes sections in order
│   └── globals.css
├── components/sections/
│   ├── Hero.tsx                     // data-vibe-id="hero"
│   ├── LogoCloud.tsx                // data-vibe-id="logos"
│   ├── Features.tsx                 // data-vibe-id="features"
│   ├── HowItWorks.tsx               // data-vibe-id="how-it-works"
│   ├── Pricing.tsx                  // data-vibe-id="pricing"
│   ├── Testimonials.tsx             // data-vibe-id="testimonials"
│   ├── FAQ.tsx                      // data-vibe-id="faq"
│   ├── CTA.tsx                      // data-vibe-id="cta"
│   └── Footer.tsx                   // data-vibe-id="footer"
├── lib/site-config.ts               // colors, fonts, copy strings
└── ...
```

Each section component reads from `site-config.ts` so the Builder can fill content **without touching layout JSX**. This is critical for keeping AI changes safe.

### Builder responsibilities for this template
- Pick which optional sections to include based on `wizard.features`
- Generate copy strings (title, subtitle, feature names, etc.)
- Generate icons (use Lucide names from a known set, never custom SVG)
- Generate a default color if user didn't pick one (default violet)
- **Never** modify the layout JSX

### What Builder does NOT do
- Add new components or sections beyond the registry
- Modify routing
- Modify build config

---

## Template 2 — Portfolio

### Goal
A personal site for creators (designers, devs, freelancers). Showcases projects.

### Pages
- **Home** — hero + about + project grid + skills + contact
- **Project detail** (`/projects/[slug]`) — full project description with images

### Sections (Home)
1. Hero (name, tagline, photo optional)
2. About (1 paragraph + skills chips)
3. Projects grid (3+ cards)
4. Contact form (name, email, message → email via Resend)

### DB tables
```sql
create table projects_data (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null,             -- the app's owner (sites can only have 1 user, but DRY)
  slug text unique not null,
  title text not null,
  description text,
  cover_url text,
  body_md text,
  is_published boolean default true,
  sort_order int default 0,
  created_at timestamptz default now()
);

create table contact_messages (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  email text not null,
  message text not null,
  created_at timestamptz default now()
);
```

### Variants
- **Minimal** — type-led, single column
- **Grid** — 2-column hero with photo, projects in masonry
- **Editorial** — magazine-style, big serif headings

---

## Template 3 — Blog

### Pages
- **Home** — header, posts list, sidebar (categories)
- **Post detail** (`/posts/[slug]`)
- **Subscribe page**

### DB tables
```sql
create table posts (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  title text not null,
  excerpt text,
  body_md text,
  cover_url text,
  category text,
  is_published boolean default false,
  published_at timestamptz,
  created_at timestamptz default now()
);

create table subscribers (
  id uuid primary key default gen_random_uuid(),
  email text unique not null,
  created_at timestamptz default now()
);
```

### Variants
- **Magazine** — grid of cards
- **Editorial** — single column, type-heavy
- **Aggregator** — dense list, like Hacker News

---

## Template 4 — To-do app

### Pages
- **Sign in / sign up** (Supabase auth on the generated app's own Supabase)
- **App** — list of todos for the signed-in user
- **Settings**

### DB tables
```sql
create table todos (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null,
  title text not null,
  is_done boolean default false,
  due_at timestamptz,
  created_at timestamptz default now()
);
-- with RLS so each user sees only their todos
```

### Variants
- **Simple list** — minimal, single list
- **Project boards** — group todos by project (extra table)

### Why this template matters
First template that uses **per-app auth and DB**, so it's the test case for our isolation strategy (ADR-002).

---

## Template 5 — Booking

### Pages
- **Public** — service list, calendar, booking form
- **Owner dashboard** — view bookings, manage services

### DB tables
```sql
create table services (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  duration_minutes int not null,
  price_usd_cents int not null,
  is_active boolean default true
);

create table bookings (
  id uuid primary key default gen_random_uuid(),
  service_id uuid not null references services,
  customer_name text not null,
  customer_email text not null,
  starts_at timestamptz not null,
  ends_at timestamptz not null,
  status text default 'confirmed',
  created_at timestamptz default now()
);
```

### Variants
- **Single service** (e.g., a freelancer with 1 type of consultation)
- **Service catalog** (multi-service salon / clinic style)

---

## Template development principles

### 1. Layout is locked, content is free (in Smart Mode)
Builder Agent never edits JSX layout files **in Smart Mode** (the default). Content lives in `site-config.ts` (or DB for dynamic templates). This makes Builder's job small + safe.

**Escape hatch — Studio Mode:** Users who need layout changes can switch a project to Studio Mode and pay per-action credits. See `docs/design/studio-mode.md`. Templates do not need any special accommodation for Studio Mode — the Studio Agent works directly on the file map.

### 2. Lucide-only icons
Builder picks icon names from a curated whitelist (~80 names). Never generates SVG. Keeps output deterministic.

### 3. Tailwind utilities only, no custom CSS
All styling via Tailwind classes. Builder may swap classes (e.g., color tokens) but doesn't write CSS.

### 4. Type-safe content
`site-config.ts` is fully typed. If Builder produces wrong shape → TypeScript catches at scaffold time, we retry once.

### 5. Mobile-first
Every variant must look good on mobile. Templates ship with proper responsive classes from the start.

### 6. Accessibility built in
- Semantic HTML (`<header>`, `<main>`, `<section>`, `<footer>`)
- ARIA labels on interactive elements
- Color contrast WCAG AA on default theme

---

## Where templates live in the repo

```
packages/templates/
├── package.json
├── src/
│   ├── types.ts                  // TemplateManifest, FileMap, …
│   ├── registry.ts               // exports all manifests
│   ├── landing/
│   │   ├── manifest.ts
│   │   ├── scaffold.ts
│   │   ├── files/                // the actual scaffold files (ts/tsx/css)
│   │   ├── variants/
│   │   │   ├── centered/
│   │   │   ├── split/
│   │   │   └── gradient-bg/
│   │   └── thumb.png
│   ├── portfolio/...
│   ├── blog/...
│   ├── todo/...
│   └── booking/...
└── tests/
```

The Builder Agent imports `registry.ts`, looks up `manifest[wizard.templateId]`, calls `scaffold(wizard)`, then refines content in the resulting file map.

---

## Open questions
- [ ] Generated apps' DB: per-app Supabase project vs single shared with RLS — see ADR-002 (M4 decision).
- [ ] How do we let creators submit new templates to the public registry? (Phase 2 marketplace feature.)
- [ ] Localization: English-only at MVP. CZ template variants in phase 2.
