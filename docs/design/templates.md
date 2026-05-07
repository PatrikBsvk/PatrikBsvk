# Templates — Starter Scaffolds

**Status:** v0.1
**Why this matters:** Templates are the **rails** that make Vibell ship reliable apps. Builder fills content, but doesn't restructure layouts. Less surface for AI to mess up = higher quality + fewer tokens.

---

## Template registry — Phase 1 MVP set (5 templates)

> **Scope (2026-05-06 v0.4):** Templates serve **non-programmers with a vision who want to build an app or website**. Audience is broad (drobní podnikatelé, creators, freelancers, students, hobbyists, indie hackers). Output is narrow (web / one-pager / simple app). Universal templates over niche-specific ones — keep options open for any vision.

| ID | Name | Best for | DB needed | Variants | First built |
|---|---|---|---|---|---|
| `landing` | Landing page (one-pager) | Launching anything: product, service, idea, event, cause | No | 3 | **Yes (M2)** |
| `multi-page` | Multi-page website | Anyone needing more than one page (about, services, contact, FAQ, gallery, hours, location) | Yes (small — content, contact messages) | 3 | M2 |
| `portfolio` | Portfolio | Designers, photographers, writers, freelancers, students showing work | Yes (small — projects, contact) | 3 | M3 |
| `booking` | Booking | Coaches, salons, fitness, advisory — anyone time-slot based | Yes | 2 | M3 |
| `shop-onepager` | Shop one-pager | Selling 1–10 products with Stripe Checkout | Yes (products, orders) | 2 | M3 |

After Phase 1 validates with paying users, we expand. Phase 2 reintroduces niche templates (Local Business, Service Business, Blog, etc.) and adds simple-app templates (auth+CRUD, dashboards) when Creator Hub ships.

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

## Template 2 — Multi-page website

### Goal
A flexible **multi-page site** for anyone whose vision needs more than a single page. Universal — works for a small business, a side project, a community, an event, a portfolio expansion, an artist's site, a non-profit. The Builder chooses which pages to scaffold based on the wizard answers.

### Pages (Builder picks 3–6 from)
- **Home** — hero + featured content + CTA
- **About** — story / mission / team
- **Services / Offerings / Menu** — list of what's offered
- **Work / Gallery / Cases** — visual showcase
- **Contact** — form + location/map (optional) + hours (optional)
- **FAQ** — common questions
- **Pricing** — if applicable
- **Blog index + post detail** — only if user explicitly enables blogging in wizard

### Configurable modules (turned on/off per project)
- Opening hours block (with "open now" indicator) — for places visitors physically visit
- Map embed (Google / Mapy.cz / OpenStreetMap)
- Newsletter signup → Resend audience
- Contact form → email via Resend
- Social links footer

### DB tables (small, generic)
```sql
create table content_pages (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  title text not null,
  body_md text,
  is_published boolean default true,
  sort_order int default 0,
  created_at timestamptz default now()
);
create table content_items (
  id uuid primary key default gen_random_uuid(),
  collection text not null,         -- 'services' | 'gallery' | 'team' | …
  data jsonb not null,              -- flexible per collection
  sort_order int default 0,
  is_published boolean default true,
  created_at timestamptz default now()
);
create table contact_messages (
  id uuid primary key default gen_random_uuid(),
  name text not null, email text, phone text, message text not null,
  created_at timestamptz default now()
);
```

### Variants
- **Editorial** — type-led, magazine feel
- **Clean** — minimal, generous whitespace
- **Warm** — rich imagery, warm palette

---

## Template 3 — Portfolio

### Goal
A **personal site for someone showing work** — designers, photographers, writers, developers, students, hobby creators, freelancers. Single primary purpose: visitors leave thinking "this person is great, I want to talk to them".

### Pages
- **Home** — hero + featured projects + about strip + contact CTA
- **Project detail** (`/work/[slug]`) — full project with images, description, links

### Sections (Home)
- Hero (name, tagline, optional photo)
- Featured work grid (3–9 items)
- About strip (1 paragraph + skills/tools chips)
- Contact form OR direct email/social links

### DB tables (small)
```sql
create table portfolio_projects (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  title text not null,
  short_description text,
  body_md text,
  cover_url text,
  links jsonb default '[]'::jsonb,        -- [{ label, url }]
  tags text[] default '{}',
  is_published boolean default true,
  sort_order int default 0,
  created_at timestamptz default now()
);

create table contact_messages (
  id uuid primary key default gen_random_uuid(),
  name text not null, email text not null, message text not null,
  created_at timestamptz default now()
);
```

### Variants
- **Minimal** — type-led, single column, big whitespace
- **Grid** — projects in masonry, photo prominent
- **Editorial** — magazine-style, big serif headings

---

## Template 4 — Booking

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

## Template 5 — Shop one-pager

### Goal
For **drobní e-shopáři** with 1–10 products who want a single page that sells. Stripe Checkout out of the box. No multi-page catalog complexity, no inventory headaches. Just product cards and "Buy now".

### Pages
- **Home** — hero + 1–10 product cards + about + reviews + footer
- **Order success** (`/success?session_id=...`) — Stripe-style thank-you
- **Order canceled** — friendly retry

### Sections
- Hero with hero product or brand statement
- Product grid (cards with image, title, short, price, "Buy" button → Stripe Checkout)
- About / story (one paragraph)
- Reviews (3 testimonials with star rating)
- Shipping & returns (collapsible)
- Footer with NAP + social

### DB tables
```sql
create table products_data (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  name text not null,
  short_description text,
  description_md text,
  price_cents int not null,
  currency text not null default 'czk',
  stripe_price_id text,           -- created via Stripe Connect when product is added
  cover_url text,
  inventory int default 0,        -- nullable means unlimited
  is_active boolean default true,
  sort_order int default 0,
  created_at timestamptz default now()
);

create table orders (
  id uuid primary key default gen_random_uuid(),
  stripe_session_id text unique,
  customer_email text,
  customer_name text,
  shipping_address jsonb,
  total_cents int not null,
  status text not null default 'pending',  -- 'pending' | 'paid' | 'fulfilled' | 'refunded'
  created_at timestamptz default now()
);
```

### Variants
- **Single product spotlight** — one hero product, big imagery
- **Multi-product grid** — up to 10 products in a responsive grid

### Why this template matters
First template that actually **moves money for the SMB**. We use Stripe Checkout (not a custom cart) to keep it dead simple and PCI-safe. Vibell handles Stripe Connect onboarding behind the scenes — the SMB never touches a developer dashboard.

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
