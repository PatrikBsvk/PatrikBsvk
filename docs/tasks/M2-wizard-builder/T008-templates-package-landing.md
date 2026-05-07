# T008 — Templates package + Landing template scaffold

**Milestone:** M2 — Wizard + Builder
**Status:** Ready
**Estimate:** 4–5 h
**Dependencies:** T001 (monorepo)

---

## Context
The Builder Agent (T009) needs **rails** to follow. Rails are templates: pre-built Next.js scaffolds with locked layouts and content placeholders. Builder fills content into these scaffolds — it never invents structure. This makes outputs reliable + cheap (small prompts, cached system messages).

This task creates the `packages/templates` workspace package and ships the **Landing template** as the first concrete one (3 variants, 8 section components, manifest, scaffold function).

**Reference docs:**
- `docs/design/templates.md` (CRITICAL — full registry spec, Landing details)
- `docs/design/studio-mode.md` (escape hatch, doesn't apply to scaffold itself)
- `docs/product/brand.md` (default theme tokens)

## Goal
A working `@vibell/templates` package with a manifest registry, type definitions, and a complete Landing template scaffold (centered / split / gradient-bg variants). The scaffold function returns a typed FileMap that compiles cleanly when written to disk.

## Acceptance Criteria
- [ ] New workspace package `packages/templates` with `package.json`, `tsconfig.json`
- [ ] Exports: `manifests`, `getTemplate(id)`, types `TemplateManifest`, `FileMap`, `WizardPayload` (re-exported from shared types)
- [ ] `src/types.ts` — full TS types matching `docs/design/templates.md` manifest format
- [ ] `src/registry.ts` — exports an array of manifests (initially: just Landing, with placeholders for the other 4)
- [ ] `src/landing/manifest.ts` — Landing manifest per `docs/design/templates.md` §"Template 1"
- [ ] `src/landing/scaffold.ts` — `scaffold(payload: WizardPayload): FileMap` function
- [ ] `src/landing/files/` — base scaffold files (TSX, CSS, config) for the locked layout
- [ ] `src/landing/variants/{centered,split,gradient-bg}/` — variant-specific overrides for `Hero.tsx`
- [ ] All section components have `data-vibe-id` attributes for click-to-edit (`hero`, `logos`, `features`, `how-it-works`, `pricing`, `testimonials`, `faq`, `cta`, `footer`)
- [ ] Each section reads content from `src/lib/site-config.ts` (typed export). Builder never modifies JSX, only the config.
- [ ] **Compile test:** a `tests/scaffold.test.ts` writes the FileMap to a temp dir, runs `pnpm install` + `pnpm build` against it, asserts success
- [ ] **Type test:** typecheck the scaffolded project (must pass strict TS)
- [ ] **Visual test (manual ok for MVP):** scaffold once with a sample payload, run the resulting Next.js project, open in browser, verify each variant renders correctly with placeholder content
- [ ] No emojis anywhere in scaffold output
- [ ] All Tailwind classes use brand tokens (violet primary by default)
- [ ] Lucide icons only (whitelist of ~30 icon names allowed for content)

## Files to create
```
packages/templates/
├── package.json
├── tsconfig.json
├── README.md  (exception to no-README rule: dev docs for adding new templates)
├── src/
│   ├── index.ts                    // public exports
│   ├── types.ts                    // TemplateManifest, FileMap, WizardPayload (re-export)
│   ├── registry.ts                 // exports [landingManifest, ...]
│   ├── shared/
│   │   ├── icons.ts                // whitelist of allowed Lucide icons
│   │   ├── slug.ts                 // slug helper (shared with apps/web)
│   │   └── tailwind-base.ts        // shared tailwind config bits
│   └── landing/
│       ├── manifest.ts
│       ├── scaffold.ts             // assembles FileMap from variant + payload
│       ├── files/
│       │   ├── package.json
│       │   ├── tsconfig.json
│       │   ├── next.config.ts
│       │   ├── tailwind.config.ts
│       │   ├── postcss.config.js
│       │   ├── src/
│       │   │   ├── app/
│       │   │   │   ├── layout.tsx
│       │   │   │   ├── page.tsx           // composes sections in order
│       │   │   │   └── globals.css
│       │   │   ├── components/sections/
│       │   │   │   ├── Hero.tsx           // base; variants override
│       │   │   │   ├── LogoCloud.tsx
│       │   │   │   ├── Features.tsx
│       │   │   │   ├── HowItWorks.tsx
│       │   │   │   ├── Pricing.tsx
│       │   │   │   ├── Testimonials.tsx
│       │   │   │   ├── FAQ.tsx
│       │   │   │   ├── CTA.tsx
│       │   │   │   └── Footer.tsx
│       │   │   └── lib/
│       │   │       └── site-config.ts     // typed content
│       │   └── public/
│       │       └── favicon.ico
│       └── variants/
│           ├── centered/
│           │   └── Hero.tsx
│           ├── split/
│           │   └── Hero.tsx
│           └── gradient-bg/
│               └── Hero.tsx
└── tests/
    ├── scaffold.test.ts             // writes FileMap, runs pnpm build
    └── landing.fixture.ts           // sample WizardPayload
```

## Type definitions (key shapes)

```ts
// packages/templates/src/types.ts

export type FileMap = Record<string /* path */, string /* content */>;

export interface TemplateSection {
  id: string;                       // 'hero', 'features', ...
  required: boolean;
  configurable: string[];           // which fields the Builder can fill
}

export interface TemplateManifest {
  id: 'landing' | 'portfolio' | 'blog' | 'todo' | 'booking';
  displayName: string;
  description: string;
  category: 'business' | 'creative' | 'personal' | 'tools' | 'education';
  thumbnail: string;
  estimatedBuildSeconds: number;
  variants: string[];
  sections: TemplateSection[];
  features: string[];                // feature chips for wizard
  databaseSchema: string[];          // SQL statements (empty for landing)
  scaffold: (payload: WizardPayload) => Promise<FileMap>;
}

export interface WizardPayload {
  templateId: TemplateManifest['id'] | 'blank';
  name: string;
  slug: string;
  description: string;
  audience: string[];
  features: string[];
  layout: { variantId: string; regenerated: boolean };
  style: {
    preset: 'minimal' | 'bold' | 'playful';
    customColor?: string;
    customRadius?: number;
    customFont?: 'geist' | 'inter' | 'space-grotesk';
  };
}
```

## site-config.ts shape (what Builder writes content into)

```ts
// generated apps/.../src/lib/site-config.ts
export const siteConfig = {
  meta: {
    title: 'Sushi Master',
    description: 'Online ordering for the best sushi in town.',
  },
  theme: {
    primary: '#7C3AED',
    radius: 12,
    font: 'geist',
  },
  sections: {
    hero: {
      enabled: true,
      title: 'The best sushi, delivered.',
      subtitle: 'Order in 30 seconds. Fresh in 30 minutes.',
      ctaPrimary: { label: 'Order now', href: '#pricing' },
      ctaSecondary: { label: 'See menu', href: '#features' },
    },
    logos: {
      enabled: false,
      prefix: 'Trusted by',
      items: [],
    },
    features: {
      enabled: true,
      title: 'Why our sushi',
      items: [
        { icon: 'Fish',     title: 'Fresh daily',  body: 'Caught at dawn.' },
        { icon: 'Clock',    title: 'Fast',         body: 'Under 30 minutes.' },
        { icon: 'Heart',    title: 'Made with love', body: 'Family recipes.' },
      ],
    },
    pricing: { enabled: true, title: 'Menu', tiers: [/* … */] },
    faq: { enabled: true, title: 'Questions', items: [/* … */] },
    cta: { enabled: true, title: 'Ready?', cta: { label: 'Order now', href: '#' } },
    footer: { enabled: true, links: [/* … */] },
  },
} as const;
```

This is the **only file Builder writes**. Layout files are scaffold-time only.

## Section component rules
- Read from `siteConfig.sections.<id>`
- If `enabled === false`, return `null`
- Always wrap top-level element with `<section data-vibe-id="<id>">`
- Use Tailwind classes derived from `theme.primary` via CSS variables

Example (`Features.tsx`):
```tsx
import { siteConfig } from '@/lib/site-config';
import * as Lucide from 'lucide-react';
import type { LucideIcon } from 'lucide-react';

export function Features() {
  const cfg = siteConfig.sections.features;
  if (!cfg.enabled) return null;
  return (
    <section data-vibe-id="features" className="py-24 px-6 max-w-6xl mx-auto">
      <h2 className="text-3xl font-semibold tracking-tight">{cfg.title}</h2>
      <div className="mt-12 grid md:grid-cols-3 gap-8">
        {cfg.items.map((item, i) => {
          const Icon = (Lucide[item.icon as keyof typeof Lucide] as LucideIcon) ?? Lucide.Sparkles;
          return (
            <div key={i} data-vibe-id={`feature-${i}`} className="rounded-xl border p-6">
              <Icon className="w-6 h-6 text-[var(--primary)]" strokeWidth={1.5} />
              <h3 className="mt-4 font-semibold">{item.title}</h3>
              <p className="mt-2 text-[var(--fg-muted)]">{item.body}</p>
            </div>
          );
        })}
      </div>
    </section>
  );
}
```

The Builder Agent (T009) will produce `siteConfig` JSON. **It will NOT touch this component file.**

## Scaffold function (skeleton)

```ts
// packages/templates/src/landing/scaffold.ts
import type { FileMap, WizardPayload } from '../types';
import baseFiles from './files';                // bundled at build time
import centeredHero from './variants/centered/Hero';
import splitHero from './variants/split/Hero';
import gradientHero from './variants/gradient-bg/Hero';
import { buildSiteConfig } from './site-config-builder';

const VARIANT_HERO: Record<string, string> = {
  centered: centeredHero,
  split: splitHero,
  'gradient-bg': gradientHero,
};

export async function scaffold(payload: WizardPayload): Promise<FileMap> {
  const files: FileMap = { ...baseFiles };

  // Override Hero with chosen variant
  const heroPath = 'src/components/sections/Hero.tsx';
  files[heroPath] = VARIANT_HERO[payload.layout.variantId] ?? VARIANT_HERO.centered;

  // Builder will overwrite site-config.ts later. For now we put a placeholder
  // built from the wizard payload so the project compiles standalone.
  files['src/lib/site-config.ts'] = buildSiteConfig(payload);

  // Inject project name into package.json
  files['package.json'] = JSON.stringify(
    {
      ...JSON.parse(baseFiles['package.json']),
      name: payload.slug,
    },
    null, 2
  );

  return files;
}
```

`buildSiteConfig(payload)` produces a sensible placeholder `siteConfig` from the wizard data. Builder Agent in T009 then improves it (better copy, real testimonials, FAQ, etc.).

## Implementation Notes
- Bundle `files/**` content using a small build-time script (`scripts/embed-files.ts`) that reads the directory and produces a TS module exporting `Record<string, string>`. Add to `pnpm build` of the package.
- Use `vitest` for the test suite.
- The compile test (`tests/scaffold.test.ts`) is gated behind a `PNPM_OFFLINE=1` env var to avoid running it in CI without internet — local dev runs it.
- Add a small CLI: `pnpm dlx @vibell/templates scaffold landing --output ./tmp/test --variant centered` for manual testing. (Optional bonus.)

## Agent Prompt (copy-paste this)

```
You are a senior engineer working on Vibell.

Read first (in order):
- docs/design/templates.md (CRITICAL)
- docs/design/studio-mode.md (escape-hatch context, no implementation here)
- docs/product/brand.md (default theme tokens)
- docs/tasks/M2-wizard-builder/T008-templates-package-landing.md (this task)

Task T008: build packages/templates with the Landing template (3 variants).

Critical rules:
- Layout JSX is locked: section components NEVER get rewritten by AI. They read from src/lib/site-config.ts (the only file the Builder Agent writes).
- All section components must wrap their top element with data-vibe-id="<id>".
- Lucide icons only (use the whitelist in shared/icons.ts).
- Tailwind classes only — no custom CSS.
- Strict TypeScript everywhere.
- No emojis.

Do this:
1. Create packages/templates workspace package with build script.
2. Implement types.ts, registry.ts, shared/ helpers.
3. Build the Landing template:
   - manifest.ts per docs/design/templates.md
   - 9 section components (Hero, LogoCloud, Features, HowItWorks, Pricing, Testimonials, FAQ, CTA, Footer)
   - 3 Hero variants (centered, split, gradient-bg)
   - site-config.ts shape (typed) and a buildSiteConfig helper
   - scaffold(payload) function that returns a complete FileMap
4. Add tests:
   - landing.fixture.ts with a sample WizardPayload
   - scaffold.test.ts that writes the FileMap to a temp dir and runs pnpm install + pnpm build (assert success)
5. Add a small CLI command to scaffold a sample project for manual visual review.

When done:
- Update task file Status: Done
- Commit: task(T008): templates package + Landing scaffold with 3 variants
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All criteria checked
- Test suite green (compile test passes when run)
- Manual visual review of all 3 variants done
- Status updated, committed, pushed
