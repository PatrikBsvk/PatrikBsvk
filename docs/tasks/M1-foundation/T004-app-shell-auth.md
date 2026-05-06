# T004 — App shell, auth UI, protected routes

**Milestone:** M1 — Foundation
**Status:** Ready
**Estimate:** 2–3 h
**Dependencies:** T001, T002, T003

---

## Context
Now that we have Next.js + Supabase + the schema, we need the user-facing shell: a layout with nav, sign-in / sign-up flows, and the concept of protected routes. This unlocks every later feature.

The shell must reflect the brand bible (violet/black premium, Geist, dark mode first-class).

**Reference docs:**
- `docs/product/brand.md` (palette, type, components)
- `docs/architecture/overview.md` (repo layout)

## Goal
A user can register, log in, log out. Logged-in user sees `/app/*` (dashboard, settings); logged-out user is bounced to `/sign-in`. Marketing routes (`/`, `/pricing`, `/about`) remain public.

## Acceptance Criteria
- [ ] Brand tokens added to `apps/web/src/app/globals.css` (violet palette, ink scale, gradients) as CSS variables
- [ ] Tailwind config maps brand tokens to Tailwind theme (e.g., `bg-violet-600`, `text-ink-1`)
- [ ] Geist Sans + Geist Mono loaded via `geist/font` package
- [ ] `<Providers>` wrapper for Supabase session + theme
- [ ] **Public layout** (`apps/web/src/app/(marketing)/layout.tsx`) — minimal top nav + logo wordmark + "Sign in" link
- [ ] **App layout** (`apps/web/src/app/(app)/layout.tsx`) — sidebar nav (Dashboard, New project, Marketplace, Settings) + user avatar dropdown
- [ ] `/sign-in` and `/sign-up` pages — email/password + OAuth (Google + GitHub) via Supabase
- [ ] `/sign-in/callback` route handler for OAuth redirect (`@supabase/ssr` cookie wiring)
- [ ] Middleware (`apps/web/src/middleware.ts`) protects `/app/*`: unauthenticated → redirect `/sign-in?next=...`
- [ ] `/app/dashboard` is the empty post-login page (just "Welcome, {name}")
- [ ] `/sign-out` action signs the user out and redirects to `/`
- [ ] `pnpm typecheck`, `pnpm build`, `pnpm dev` all pass
- [ ] Visually verified: light mode + dark mode look like the brand bible. Premium hero CTA uses violet gradient.

## Files to create / modify
```
apps/web/src/
├── app/
│   ├── (marketing)/
│   │   ├── layout.tsx
│   │   └── page.tsx                  // simple "Coming soon" hero — proper landing in T007
│   ├── (auth)/
│   │   ├── sign-in/page.tsx
│   │   ├── sign-up/page.tsx
│   │   └── callback/route.ts         // Supabase OAuth callback
│   ├── (app)/
│   │   ├── layout.tsx
│   │   ├── dashboard/page.tsx
│   │   └── settings/page.tsx         // empty stub
│   └── globals.css                   // brand tokens
├── components/
│   ├── providers.tsx
│   ├── nav/
│   │   ├── marketing-nav.tsx
│   │   └── app-sidebar.tsx
│   ├── auth/
│   │   ├── sign-in-form.tsx
│   │   └── sign-up-form.tsx
│   └── brand/
│       └── wordmark.tsx              // <Wordmark size="md" /> renders "vibell" in Geist 600
├── lib/
│   ├── auth/actions.ts               // server actions for sign-in/sign-up/sign-out
│   └── routes.ts                     // typed route helpers
└── middleware.ts                     // auth gate for /app/*
```

## Implementation Notes

### Brand tokens in `globals.css`
```css
@import "tailwindcss";
@import "geist/font/sans";
@import "geist/font/mono";

:root {
  --paper: #ffffff;
  --ink-0: #0a0a0a;
  --ink-1: #171717;
  --ink-2: #404040;
  --ink-3: #737373;
  --ink-5: #d4d4d4;
  --ink-6: #f5f5f5;
  --ink-7: #fafafa;

  --violet-400: #a78bfa;
  --violet-500: #8b5cf6;
  --violet-600: #7c3aed;
  --violet-700: #6d28d9;
  --violet-800: #5b21b6;

  --bg: var(--paper);
  --bg-surface: var(--ink-7);
  --fg: var(--ink-0);
  --fg-muted: var(--ink-2);
  --border: var(--ink-5);
  --primary: var(--violet-600);

  --radius-sm: 0.5rem;
  --radius-md: 0.75rem;
  --radius-pill: 999px;

  --shadow-card: 0 4px 12px rgba(0,0,0,0.04);
  --shadow-glow: 0 0 24px rgba(124,58,237,0.4);

  --ease: cubic-bezier(0.16, 1, 0.3, 1);
}

[data-theme="dark"] {
  --bg: #0a0a0a;
  --bg-surface: #0f0f12;
  --fg: #fafafa;
  --fg-muted: #a3a3a3;
  --border: #262626;
  --shadow-card: 0 4px 12px rgba(0,0,0,0.3);
}
```

### Theme toggle
Use `next-themes` to switch between `light` and `dark` data attribute. Default to dark on the marketing site (premium first impression), light on `/app/*` (working comfort).

### Auth UX
- Sign-in / sign-up forms styled with shadcn `Input`, `Button`, `Label`, `Card`.
- Primary CTA uses violet gradient + soft glow on dark.
- Error messages calm, no emojis (per brand bible voice rules).
- Show "Continue with Google / GitHub" buttons above the email/password form.

### Middleware
- Use `@supabase/ssr` `updateSession` pattern.
- Match `/app/:path*` only. Marketing & auth routes pass through.

### Wordmark component
```tsx
export function Wordmark({ size = "md" }: { size?: "sm" | "md" | "lg" }) {
  const fontSize = size === "sm" ? "text-lg" : size === "lg" ? "text-3xl" : "text-xl";
  return (
    <span className={`${fontSize} font-semibold tracking-[-0.04em]`}>
      vibell
    </span>
  );
}
```
The dot above the "i" can later be the violet glow accent. For MVP, plain wordmark is fine.

## Agent Prompt (copy-paste this)

```
You are a senior engineer working on Vibell.

Read first:
- docs/product/brand.md (CRITICAL — palette, type, components)
- docs/product/vision.md
- docs/architecture/overview.md
- docs/tasks/M1-foundation/T004-app-shell-auth.md (full spec)

Task T004: build the app shell with auth.

Constraints:
- Strict TypeScript
- Use @supabase/ssr for auth (cookies-based, App Router compatible)
- Use shadcn/ui for inputs, buttons, cards
- Brand: violet (#7C3AED) primary, near-black, Geist font, dark mode first-class
- Marketing routes default to dark theme; /app/* defaults to light
- Use next-themes for the toggle
- No emojis in product UI
- All copy in English (CZ localization later)

Do this:
1. Add brand tokens to globals.css.
2. Map tokens to Tailwind theme.
3. Build (marketing), (auth), (app) route groups.
4. Implement sign-in / sign-up / sign-out / OAuth callback.
5. Build the app sidebar with Dashboard / New project / Marketplace / Settings links (Marketplace + Settings can be empty stubs).
6. Add middleware that protects /app/*.
7. Verify build, typecheck, and visually check both themes match the brand bible.

When done:
- Update task file Status: Done
- Commit: task(T004): app shell with auth and brand tokens
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All criteria checked
- Visually consistent with `docs/product/brand.md`
- Both themes work, OAuth + email auth tested manually
- Status updated, committed, pushed
