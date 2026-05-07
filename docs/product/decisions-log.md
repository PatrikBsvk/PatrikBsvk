# Decisions Log — 2026-05-06 (Strategic batch #1)

> Snapshot of strategic decisions made by founder Patrik in PM session.
> Each item captures: what, founder's exact words/intent, my proposed details, status.

---

## ✅ Approved — Tier S (will become design docs + tasks)

### D-001 · Smart Autofill from URL
- **What:** User pastes a URL (LinkedIn, existing site, brand guide PDF, Notion). Agent extracts logo, palette, copy, tone, then pre-fills the wizard.
- **Founder:** *"líbí se mi to"*
- **Cost:** 3 credits per extraction
- **Why now:** removes biggest onboarding friction. 10× faster start.
- **Status:** ready for design doc

### D-002 · AI Debugger
- **What:** When build / preview / publish / Studio edit fails, agent reads error → proposes fix → with consent applies it.
- **Founder:** *"chci ho ale nechci ho platit"* → user-facing free
- **Cost:** **0 credits** for user. Vibell absorbs (~$0.10/active user/mo). Justified by retention impact.
- **Why now:** non-programmers cannot debug. Without it, first error = churn.
- **Status:** ready for design doc

### D-003 · Pre-built Integrations Library
- **What:** Catalog of integration cards in workspace (Stripe, Calendly, Mailchimp, Resend, Notion CMS, GA, Slack, etc.). Click "Enable" → integration agent wires API key, scaffolds components, tests.
- **Founder:** *"to chci být skutečný app builder"*
- **Cost:** 5–10 credits per setup (varies by complexity)
- **MVP integrations (8–10):** Stripe, Calendly, Mailchimp, Resend, Notion, Google Analytics, Slack, Discord webhook, Tally forms, Cal.com
- **Why now:** turns Vibell from site builder into app builder; new credit-spend driver
- **Status:** ready for design doc

### D-004 · AI Brand Designer (full brand bible)
- **What:** From one description, agent generates **complete brand identity** — logo, full color system, typography, voice samples, sample components — and applies it across the user's project.
- **Founder:** *"chci aby byl schopen vytvořit totální brand bible na které bude stavět aplikaci"*
- **Cost:** TBD (founder said "určíme později"). Proposal: **15–20 credits** per generation (Sonnet 4.6, possibly with image generation for logo).
- **Output:** updates project's `theme` block in `site-config.ts` + creates a new `brand_bible` table row with all assets
- **Why now:** solves "I don't have a brand" pain; magic moment that justifies premium tier
- **Status:** ready for design doc; price to confirm in design

### D-005 · Code Export + GitHub Sync (Pro+ only)
- **What:** Pro+ users can: download a clean Next.js zip (one-click) OR enable bidirectional GitHub sync (edit in Vibell or in Cursor — both stay in sync).
- **Founder:** *"chci pro placené členy"* (Pro+ tier)
- **Cost:** included in Pro plan
- **Why now:** kills #1 buyer objection — "vendor lock-in"
- **Status:** ready for design doc

### D-006 · Built-in Publish Analytics
- **What:** Every published Vibell app gets analytics out of the box: visitors, top pages, traffic sources, conversion events. Visible in workspace dashboard. AI suggestions on improvements ("your hero CTA gets 3% click — try X").
- **Founder:** *"to se mi líbí — automaticky mu postaví analytics, které si může sledovat u nás"*
- **Cost:** 0 credits for user (we run the tracker, low marginal cost). Pro+ unlocks AI-suggested improvements (~3 credits per audit).
- **Why now:** closes the loop (build → publish → measure → improve). Most builders are black box after publish — big differentiator.
- **Status:** ready for design doc

### D-007 · Voice Mode (for everyone, all plans)
- **What:** User can talk to the workspace. Coach + click-to-edit + wizard inputs all support voice. Premium accessibility + delight.
- **Founder:** *"voice mode chci pro všechny pro komunikaci s agentem"*
- **Cost:** **free** for all plans (we eat speech-to-text cost ~$0.005/min)
- **Why:** founder explicitly chose universal access — strong brand signal of "everyone can build"
- **Status:** ready for design doc

### D-008 · AI SEO Assistant
- **What:** After publish, agent audits the site (meta tags, alt text, schema.org, content depth, internal links). Suggests improvements with one-click apply.
- **Founder:** *"ai seo assistant chci"*
- **Cost:** 5 credits per audit. Pro+ gets monthly auto-audit free.
- **Status:** ready for design doc

### D-009 · Smart Image Generation
- **What:** Generate branded hero images, illustrations, product mocks. Style locked to project palette + tone.
- **Founder:** *"smart image generation ale zaplatí si to uživatel"* — **user pays**
- **Cost:** 8 credits per image (covers our cost + small margin). Pro+: 20% discount.
- **Status:** ready for design doc

### D-010 · Continuity Coach ("kde mám pokračovat")
- **What:** When user lands on dashboard or a project, AI proactively suggests next steps based on project state and Memory. *"You haven't added testimonials yet — most landing pages convert better with them. Want me to draft three?"*
- **Founder:** *"chci aby asistent mě i vedl kde mám pokračovat když nevím"*
- **Cost:** 0 credits (background Coach + Memory combo)
- **Status:** ready for design doc

### D-011 · Referral / Ambassador Program
- **What:**
  - Every Vibell user can share a referral link.
  - Ambassador earns **10% of paid membership** revenue from referrals.
  - New user (referee) gets **100 free credits** on top of Free plan's 50 credits → 150 credits month 1.
- **Founder:** *"každý může sdílet být ambasador, získá 10% z placeného členství. plus nováček 100 free kreditů"*
- **Open detail (PM proposal):**
  - Lifetime 10% **OR** capped at 12 months? **Recommend lifetime** for first 1 000 users (founding ambassadors), then re-evaluate. Simpler messaging.
  - Tracking: standard cookie + first-touch attribution
  - Payouts: via Stripe Connect, monthly minimum $20
- **Status:** needs design doc with payout flow + ambassador dashboard

### D-012 · Creator Hub — profile, articles, tips, paid membership (CONFIRMED 2026-05-06)
- **Founder clarification:** *"tvůrce má vlastní profil kde může přispívat články tipy nebo mít vlastní membership"*
- **What:** Each Vibell user has a public Creator Hub at `vibell.app/@username`. The hub is more than a portfolio — it's a content + community + monetization page. The creator can:
  - **Articles** — long-form posts (build journeys, tutorials, behind-the-scenes)
  - **Tips** — short posts (Twitter-style microcontent)
  - **Apps showcase** — their portfolio of Vibell-built apps
  - **Marketplace listings** — apps and templates for sale
  - **Paid membership** — followers pay $X/mo for access to gated articles, tips, app downloads, behind-the-scenes drops
  - **Free follow** — fans can follow without paying for the public feed
- **Mental model:** Substack + Patreon + Dribbble portfolio, all in one, baked into the Vibell ecosystem.
- **Why huge:** **deepens the Social Network pillar dramatically.** Creators don't just *build* on Vibell — they *grow an audience and earn* on Vibell. Network effects: more creators → more followers → more incoming creators. Lock-in is healthy: the creator's audience lives on Vibell.
- **Vibell platform fee on creator memberships:**
  - Free + Basic: 15 % of MRR
  - Standard: 12 %
  - Pro: 10 %
  - Team: 7 %
  - (Stripe fees on top, paid by the creator's gross — standard.)
- **Setup cost for creator:** 10 credits (Membership Agent wires Stripe Connect, creates tiers, gates content).
- **Cost per article published:** 0 credits (just text, free). AI-assist for writing articles: 2 credits per draft.
- **Why this beats base44 / Lovable / Bolt by miles:** none of them have a creator-economy layer. This is a moat that compounds over time.

### D-013 · Membership inside user-built apps — DEFERRED
- Originally I (PM) interpreted D-012 as "paid memberships inside user-built apps" (like Memberful for Vibell apps). Founder clarified the intent was the Creator Hub (D-012 corrected).
- **Memberships inside user-built apps** is still a strong product idea — every Vibell-built app could have its own paid tiers — but it's now **a separate future feature, not Phase 2**.
- **Status:** parked in `ideas-backlog.md` as Tier A for later phase

---

## ⏸ On hold — explicit defer per founder

Tier A items deferred per: *"na začátku ne, teď se zaměříme na budování studia produktu, pak na uživatel může prodávat..."*

Phase ordering, founder-stated:
1. **Studio (current)** — Wizard + Builder + Workspace + Click-to-edit + Studio Mode + Mockup Library
2. **Marketplace + portfolio + memberships (next)** — sell apps, public profile pages, paid memberships in user apps
3. **Polish & growth (after)** — referral, SEO, debugger, voice, etc.

Items on hold for Phase 1 → designed in Phase 2:
- Component Shop (A1)
- Vibell Showcase + featured slots (A2)
- Custom domains UI (A5) — implicit in marketplace phase

---

## ❌ Rejected / not now

(none this batch)

---

## 🛣 PM next steps

Based on these decisions, design docs queue (in priority order):

1. **D-012 Creator Hub** (profile + articles + tips + paid membership) — biggest strategic moat, drives Phase 2
2. **D-004 AI Brand Designer** — flagship magic moment, drives premium upgrades
3. **D-001 Smart Autofill from URL** — quick onboarding win
4. **D-002 AI Debugger** — retention insurance
5. **D-003 Pre-built Integrations** — turns site builder into app builder
6. **D-006 Built-in Analytics** — closes the build→measure loop
7. **D-011 Referral program** — viral growth lever
8. **D-005 GitHub Sync (Pro+)** — kills lock-in objection
9. **D-007 Voice Mode** — universal delight (all plans)
10. **D-008 SEO Assistant**
11. **D-009 Image Generation**
12. **D-010 Continuity Coach**

Phase mapping:
- **Phase 1 (current, M2-M3):** Studio product (Wizard, Builder, Workspace, Click-to-edit, Studio Mode, Mockup Library)
- **Phase 2 (M4-M5):** Creator Hub + Marketplace + memberships, Brand Designer, Autofill, Debugger, Analytics, Integrations
- **Phase 3 (M6+):** Referral, Voice, SEO, Image gen, GitHub sync, Continuity Coach

Each design doc will include implementation tasks scoped to fit the active milestone.
