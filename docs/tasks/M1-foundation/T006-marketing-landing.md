# T006 — Marketing landing page (vibell.app)

**Milestone:** M1 — Foundation
**Status:** Ready
**Estimate:** 3–4 h
**Dependencies:** T004 (brand tokens, app shell)

---

## Context
Vibell needs a landing page that does three jobs:
1. **Convert** visitors to sign-ups (one obvious CTA).
2. **Showcase the brand** — first impression must scream "premium violet/black".
3. **Eat its own dog food** — the layout and tone match what users will build with the Landing template.

Founder direction (chosen): Stripe-vibe headline — *"Your idea. A real URL. 30 minutes."*

**Reference docs:**
- `docs/product/brand.md` (CRITICAL — palette, type, voice)
- `docs/product/competition.md` (positioning, two-pillar moat)
- `docs/product/vision.md`
- `docs/design/ux-flows.md` (entry into the "Wow" path)

## Goal
A polished marketing landing page at `vibell.app/` that introduces Vibell, leads to sign-up, and renders perfectly in dark mode (default) and light mode.

## Acceptance Criteria
- [ ] Route: `/` in `(marketing)` group, dark theme by default
- [ ] Hero with headline + subheadline + 2 CTAs (Get started / See examples)
- [ ] Section: "Two reasons Vibell is different" — Guide System + Social Network (the moat)
- [ ] Section: "How it works" — 4 steps (Pick template → Tell us → Click to edit → Publish)
- [ ] Section: Live demo strip — animated mockup of click-to-edit interaction (CSS-only animation, no real iframe)
- [ ] Section: Pricing — 4 plans (Free / Basic / Standard / Pro) with credit counts; Team teased ("contact us")
- [ ] Section: FAQ — 6 questions
- [ ] Section: Final CTA — full-bleed dark gradient, "Have an idea. Vibell builds it." + sign-up button
- [ ] Footer — wordmark, link columns (Product / Company / Legal), social, copyright
- [ ] All copy in English
- [ ] All buttons / links route correctly (Get started → /sign-up)
- [ ] Lighthouse: Performance ≥ 90, Accessibility ≥ 95, Best Practices ≥ 95, SEO ≥ 95
- [ ] OpenGraph image (`/og.png`) and proper meta tags
- [ ] Light mode works and is equally polished
- [ ] No emojis in any copy

## Copy (final, do not "improve")

### Hero
- **Headline:** *Your idea. A real URL. 30 minutes.*
- **Subheadline:** *Vibell turns ideas into published web apps. Without code. Without prompts. Without DevOps.*
- **Primary CTA:** *Get started free*
- **Secondary CTA:** *See live examples*
- **Below CTAs:** *Free plan includes 50 credits — no credit card.*

### Two-pillar section
- **Section title:** *Built different. Built for you.*
- **Card 1 (Guide System)**
  - *A guide, not a prompt box.*
  - Body: *Pick a template. Answer four questions. Click to edit. You're never staring at a blank screen.*
- **Card 2 (Social Network)**
  - *Your portfolio. Your store.*
  - Body: *Every Vibell creator gets a public profile at vibell.app/@you. Publish, share, sell. Earn 70 to 80 percent on every clone.*

### How it works (4 steps)
1. *Pick a template* — *20 starting points. Or describe your idea and we'll match one.*
2. *Tell us about it* — *Name, audience, features. Voice input or one-click AI fill.*
3. *Click to edit* — *Tap any element. Change text, color, or describe a change. We show you three options.*
4. *Publish* — *One click. Live at yourapp.vibell.app. Custom domain on Pro.*

### Pricing
| Plan | Price | Credits | Headline benefit |
|---|---|---|---|
| Free | $0 | 50 / mo | Try it, ship one app |
| Basic | $15 / mo | 200 / mo | No watermark, custom subdomain |
| Standard | $39 / mo | 1 000 / mo | Priority queue, more storage |
| Pro | $99 / mo | 3 000 / mo | Custom domain, 80% marketplace split |
| Team | Contact | 9 000+ | 5 seats, collaboration |

### FAQ
1. *Do I need to know how to code?* — No. Vibell is built for non-programmers. If you've used Notion or Canva, you're ready.
2. *What kinds of apps can I build?* — Landing pages, portfolios, blogs, to-do apps, booking systems. More templates ship every month.
3. *Where are my apps hosted?* — On Vibell's infrastructure. We handle hosting, databases, and SSL. Free apps run on shared subdomains; paid plans get custom domains.
4. *Can I export my code?* — Yes. Pro plan includes one-click export to GitHub.
5. *What's a credit?* — A unit of AI work. Most actions cost 1–5 credits. Credit costs are shown before you click.
6. *Can I sell my apps?* — Yes. List on Vibell Store and earn 70–80% per sale.

### Final CTA section
- *Have an idea. Vibell builds it.*
- *Start with 50 free credits. No credit card required.*
- Button: *Create your account*

### Footer
- Wordmark + tagline (*"AI app builder for the rest of us"*)
- **Product:** Templates · Marketplace · Pricing · Changelog
- **Company:** About · Blog · Careers · Contact
- **Legal:** Terms · Privacy · DPA · Cookies
- **Social:** X · GitHub · Discord · YouTube
- *© 2026 Vibell. Made with Vibell.*

## Files to create
```
apps/web/src/
├── app/(marketing)/
│   ├── page.tsx                              // composes sections
│   └── layout.tsx                            // dark default
├── components/marketing/
│   ├── Hero.tsx
│   ├── TwoPillars.tsx
│   ├── HowItWorks.tsx
│   ├── DemoStrip.tsx                         // animated mockup
│   ├── Pricing.tsx
│   ├── FAQ.tsx
│   ├── FinalCTA.tsx
│   └── Footer.tsx
├── components/brand/
│   └── WordmarkGlow.tsx                      // wordmark with violet glow halo
└── public/
    ├── og.png                                // 1200x630 OG image
    └── thumbs/landing.png                    // for templates page later
```

## Implementation notes

### Hero
- Full-viewport-height first fold (ish — actually 88vh, leave hint of next section).
- Background: dark hero gradient (`#0A0A0A → #1B0F3F → #2E1065`) with subtle violet noise texture.
- Headline in Geist Sans 600, `text-display`, white.
- Subheadline in Geist Sans 400, `text-body-lg`, `--ink-3`.
- Primary CTA: violet gradient + glow.
- Secondary CTA: ghost button.
- Behind text: a static abstract render of a Vibell workspace mock at 0.4 opacity (CSS only, no image).

### Demo strip section
Animated visual: a faux Vibell workspace card. Use CSS animations only (no JS). Loops:
1. Cursor enters the demo
2. Hovers over a "Hero" element → violet outline appears
3. Click effect → edit panel slides in
4. Three variant cards animate in (skeleton → filled)
5. One is selected → demo updates
6. Loop

This sells the "click to edit" concept without needing a video.

### Pricing
- 4 cards in a row on desktop, stacked on mobile.
- Most-popular highlight on Standard (subtle violet border + "Most popular" pill).
- Each card: name, price, credits, 4 bullet features, CTA button.
- Below: small note "Need more? Team plan from $259/mo — talk to us"

### Theme toggle
Top-right of marketing nav: sun/moon icon. Default dark. Persist in localStorage.

### Accessibility
- All sections have proper landmarks.
- Color contrast tested in both modes.
- Focus rings always visible (use `focus-visible` with violet ring).
- All animations respect `prefers-reduced-motion`.

### Performance
- Geist font loaded with `display: swap`.
- All images are AVIF + fallback WebP.
- No third-party scripts on first load (analytics added later via T0xx-PostHog).
- Inline critical CSS, lazy-load below-the-fold animations.

### SEO
- `<title>`: *Vibell — Your idea. A real URL. 30 minutes.*
- `<meta name="description">`: 150–160 char version of subheadline.
- OpenGraph: title, description, og:image, twitter:card=summary_large_image.

## Agent Prompt (copy-paste this)

```
You are a senior engineer + designer working on Vibell.

Read first (in this order, fully):
- docs/product/brand.md
- docs/product/vision.md
- docs/product/competition.md
- docs/design/ux-flows.md
- docs/tasks/M1-foundation/T006-marketing-landing.md (this task — copy is final)

Task T006: build the marketing landing page at vibell.app/.

Critical rules:
- Use the EXACT copy from the task spec. Do not "improve" it.
- Brand bible is law: violet (#7C3AED) primary, near-black background, Geist Sans, no emojis.
- Dark theme is the default for the marketing route. Light theme must work too.
- Use Lucide icons only.
- All animations CSS-only (no JS libraries beyond what we already have).

Do this:
1. Build all 8 sections as separate components under components/marketing/.
2. Compose them in app/(marketing)/page.tsx.
3. Set marketing layout to default to dark theme.
4. Add OG meta tags + a generated /og.png (use @vercel/og or similar).
5. Add the demo strip with CSS-only animation.
6. Verify Lighthouse hits 90+/95+/95+/95+ targets.
7. Both themes look polished (test manually).

When done:
- Update task file Status: Done
- Commit: task(T006): marketing landing page with violet hero and two-pillar moat
- Push to claude/github-repo-setup-NvGZL
```

## Definition of Done
- All criteria checked
- Lighthouse targets met
- Both themes look like the brand bible
- Final CTA links to /sign-up
- Status updated, committed, pushed
