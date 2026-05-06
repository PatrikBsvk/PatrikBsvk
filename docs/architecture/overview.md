# Vibell — Architecture Overview

## High-level layers

```
┌─────────────────────────────────────────────────────────┐
│  Vibell Web App (apps/web)                              │
│  ├─ Wizard (multi-step form)                            │
│  ├─ Workspace (preview + click-to-edit + chat)          │
│  ├─ Marketplace + portfolio                             │
│  └─ Billing & settings                                  │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  API layer (Vercel Functions, in apps/web/app/api)      │
│  ├─ /api/wizard        wizard state                     │
│  ├─ /api/agents/*      agent invocations                │
│  ├─ /api/deploy        publish to Vercel                │
│  ├─ /api/billing/*     Stripe webhooks + checkout       │
│  └─ /api/marketplace/* listings, purchases              │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┬─────────────┐
        ▼              ▼              ▼             ▼
   ┌─────────┐   ┌──────────┐   ┌─────────┐   ┌─────────┐
   │ Agents  │   │ Supabase │   │ Stripe  │   │ Vercel  │
   │ (Claude)│   │ (DB+Auth)│   │ (billing│   │   API   │
   └─────────┘   └──────────┘   └─────────┘   └─────────┘
```

## Repo layout (target)

```
PatrikBsvk/                      # repo root, claude/github-repo-setup-NvGZL
├── apps/
│   └── web/                     # Vibell main Next.js 15 app
│       ├── src/app/             # App Router pages + API routes
│       ├── src/components/      # Vibell UI
│       ├── src/lib/             # client + server helpers
│       └── ...
├── packages/
│   ├── agents/                  # Agent SDK + system prompts
│   │   ├── src/orchestrator.ts
│   │   ├── src/builder.ts
│   │   ├── src/ui-editor.ts
│   │   ├── src/variant.ts
│   │   ├── src/data-schema.ts
│   │   ├── src/deploy.ts
│   │   ├── src/coach.ts
│   │   └── src/prompts/         # cached system prompts
│   ├── ui/                      # shared shadcn components used by web
│   └── templates/               # generated-app templates (To-do, Blog, ...)
│       ├── todo/
│       ├── blog/
│       ├── portfolio/
│       ├── booking/
│       └── landing/
├── docs/                        # this documentation
├── pnpm-workspace.yaml
├── package.json
├── tsconfig.base.json
└── .gitignore
```

## Tech stack

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 15 (App Router) | SSR, RSC, file-based routing, Vercel-first |
| UI | React 19 + Tailwind 4 + shadcn/ui | Fast iteration, consistent components |
| State | Zustand + TanStack Query | Lightweight, well-typed |
| Auth | Supabase Auth | Email + OAuth, RLS for free tier |
| DB | Supabase Postgres | Postgres, RLS, generous free tier |
| Storage | Supabase Storage | For project assets |
| AI | `@anthropic-ai/sdk` (Claude) | Sonnet 4.6 + Haiku 4.5 + prompt caching |
| Payments | Stripe + Stripe Connect | Subscriptions + marketplace payouts |
| Preview | WebContainers (StackBlitz) | In-browser sandbox for generated apps |
| Deploy (user apps) | Vercel REST API | Programmatic project creation |
| Hosting (Vibell) | Vercel | Same vendor as deploy target |
| Email | Resend | Transactional, dev-friendly |
| Errors | Sentry | Frontend + serverless |
| Analytics | PostHog | Product analytics + feature flags |

## Generated apps (user output)

- **Stack (fixed):** Next.js + Tailwind + shadcn/ui + Supabase
- **Why fixed:** One stack = one set of templates = simpler agents + cheaper to maintain.
- **Hosting:** Each user app = one Vercel project under Vibell's account.
- **Subdomain:** `[slug].vibell.app` (free/basic), custom domain on Pro+.
- **Database:** Per-app Supabase project (decision pending — see ADR-002).

## Architectural Decision Records (ADRs)

ADRs live in `docs/architecture/adrs/` (created when first decision is recorded).

Pending ADRs:
- **ADR-001:** pnpm monorepo (proposed; will document with T001)
- **ADR-002:** Per-app Supabase project vs. single shared DB with RLS
- **ADR-003:** WebContainers vs. server-side preview (Vercel preview deployment)
- **ADR-004:** Vercel REST API vs. Vercel CLI for programmatic deploys
- **ADR-005:** Direct `@anthropic-ai/sdk` vs. Claude Agent SDK

Each ADR follows: Context · Decision · Consequences · Alternatives considered.
