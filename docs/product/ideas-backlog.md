# Vibell — Ideas Backlog

> **Status:** PM brainstorm of product directions beyond the agreed MVP.
> Each idea is ranked: **S** (urgent strategic), **A** (high value), **B** (worth keeping in mind), **moonshot** (cool but later).
>
> Founder picks which ideas to develop into design docs + tasks.

---

## Tier S — Strategic, develop next

### S1. AI Brand Designer
A separate guided flow that produces a **complete brand identity** (logo, color system, typography, voice samples) from one description, then applies it across the user's whole project in seconds.

- Why: solves the "I don't have a brand yet" pain. Saves hours. Feels like magic. Justifies premium tier.
- Scope: 1 new agent (Brand Designer, Sonnet 4.6), 1 new flow accessible from wizard step 4 and from workspace sidebar
- Cost (per generation): ~10 credits
- Differentiator: base44 has none of this; Lovable/Bolt make you bring your own brand

### S2. Smart Autofill from URL
User pastes any URL (their LinkedIn, existing site, brand guide PDF, Notion page) → an agent fetches it, extracts logo, palette, copy, tone, and pre-fills the wizard.

- Why: removes biggest friction in onboarding (the wizard form). 10× faster start for users with existing identity.
- Scope: 1 new agent (Extractor, Haiku 4.5 + headless screenshot worker)
- Cost: 3 credits per extraction
- Differentiator: nobody in this space does this yet (as of 2026-05)

### S3. Code export + GitHub sync (Pro+)
Pro users can: download a clean Next.js zip (one click), or **bidirectionally sync** with their GitHub repo. Edit in Vibell or in Cursor — both stay in sync.

- Why: kills the #1 buyer objection ("I don't want to be locked in"). Builds trust with savvy users.
- Scope: GitHub OAuth integration, sync agent, conflict resolution UI
- Differentiator: huge trust signal; base44 reportedly weak here

### S4. AI Debugger (build-time fixer)
When build fails (in WebContainers preview, on publish, or after a Studio edit), an agent reads the error, identifies the cause, proposes a fix, and (with consent) applies it.

- Why: non-programmers can't debug. Without this, every error becomes a support ticket.
- Scope: Debugger agent (Sonnet 4.6), wired into WebContainers + deploy pipeline
- Cost: 0 credits (we eat — it reduces churn)
- Critical for retention

### S5. Pre-built integrations (Stripe, Calendly, Mailchimp, …)
A library of integration cards in workspace sidebar. User clicks "Enable Stripe" → agent wires the API key, creates checkout components, tests it. Same for Calendly bookings, Mailchimp subscribers, Resend emails, Notion CMS, Google Analytics, etc.

- Why: makes Vibell apps actually useful, not just pretty. Recurring revenue: each integration costs credits to set up + ongoing usage credits.
- Scope: 1 integration agent, 8–10 hand-built integration cards for MVP
- Cost: 5–10 credits per integration setup
- Differentiator: turns Vibell from "site builder" into "app builder"

### S6. AI SEO Assistant
Post-publish, an agent reviews the live app and suggests improvements: meta tags, alt text, schema.org, content additions, internal links. One-click apply per suggestion.

- Why: most landing pages get this wrong, and good SEO drives recurring traffic = recurring loyalty to Vibell.
- Scope: SEO agent (Haiku 4.5), runs on demand or as a "weekly health check"
- Cost: 5 credits per audit
- Differentiator: zero competitors offer this baked in

---

## Tier A — High value, queue after S

### A1. Component Shop (Studio-only)
Curated library of pre-built component variants (hero styles, pricing tables, FAQ patterns, navbar variants, footer layouts) that Studio users drag into their project. Creators can publish their own components and sell them. Small marketplace inside Studio.

- Phase 2-3 marketplace play; trust signal in MVP

### A2. Vibell Showcase + featured slots
Homepage gallery of "Made with Vibell" apps. Curated weekly. Creators can pay credits to be featured (new revenue stream + organic marketing).

- Easy to ship; adds inspiration & social proof

### A3. Founder dashboard for users (publish analytics)
Built-in analytics on every published app: visitors, top pages, traffic sources, conversion events. AI-suggested improvements based on data.

- Closes the loop: build → publish → measure → improve. Most app builders are black boxes after publish.

### A4. Voice mode for workspace
Talk to the workspace; all click-to-edit and Coach interactions can be done by voice. Premium accessibility + delight.

- Strong delight factor, accessibility, demo-friendly. Voice agent already specced.

### A5. Custom domains UI (Pro+)
Simple input field, we handle DNS + SSL via Vercel API. No-touch for the user.

- Already in plan but worth flagging as priority for Pro upgrade conversion

### A6. Referral program
Share Vibell with a friend → both get credits when they join. Standard SaaS growth lever.

- Cheap, high impact

### A7. Image generation (hero images, illustrations)
Generate branded images for hero sections, illustrations, and product mocks via Anthropic image API or 3rd party. Style locked to project palette.

- Big visual upgrade; needs prompt safety + content moderation

### A8. Smart "Continue where you left off"
On every dashboard return, AI proactively suggests next steps based on current project state ("You haven't added testimonials yet — most landing pages convert better with them. Want me to draft three?").

- Engagement driver. Cheap (background Memory Agent + Coach combo).

---

## Tier B — Good ideas, explore later

- **B1.** Public build streams (Twitch-style watch creators build live)
- **B2.** Templates with creator backstory (humanizes marketplace)
- **B3.** Vibell forum / Discord integration baked into product
- **B4.** Follow / activity feed on dashboard
- **B5.** Collaborator invites (free guest seats on Pro)
- **B6.** Vibell credits as currency (creators paid in credits, circular economy)
- **B7.** Pre-built edge-functions for forms / emails ("when form submits, email me")
- **B8.** Templates published as npm packages (centralized updates without re-deploying every app)
- **B9.** Realtime collaboration (Team plan)
- **B10.** Plagiarism / spam detection on marketplace listings
- **B11.** "Vibell for X" SEO landing pages (landing builder, portfolio builder, blog builder, …)
- **B12.** Vibell Pulse weekly email (community newsletter)
- **B13.** AI-first onboarding video personalized per user
- **B14.** Interactive cursor pointer animation when AI is editing (visceral demo)
- **B15.** Vibell Time — timeline view of project build, shareable on socials

## Moonshots — for inspiration

- **M1.** Figma plugin to import Figma frames as Vibell mockups
- **M2.** Vibell desktop app for offline/private project work
- **M3.** Vibell mobile app — build apps on a phone (founder mentioned this for phase 3)
- **M4.** "Vibell for agencies" — white-label, sub-organize creator teams under one brand
- **M5.** Open-source community templates with contributor leaderboard

---

## PM recommendations (founder to pick from)

If we ship just **3 of these in M5–M6**, biggest wins:

1. **S2 — Smart Autofill from URL.** Cheapest moonshot. 10× onboarding magic.
2. **S4 — AI Debugger.** Without it, our retention collapses on first build error.
3. **S5 — Pre-built integrations.** Turns site builder into app builder; new credit-spend driver.

Hold for next quarter:
- S1 (AI Brand Designer) — needs design polish, can wait until template gallery is bigger
- S3 (GitHub sync) — important for Pro tier, but we don't have many Pro users yet
- S6 (SEO Assistant) — high value but harder to do well; quality bar is high

---

## How to use this file

Founder reviews and **promotes ideas** to design docs / tasks by moving them above the line. PM (Claude) then writes:
1. A design spec under `docs/design/<idea>.md`
2. Implementation tasks under `docs/tasks/Mx-*/Txxx-*.md`

Ideas can also be **demoted or removed** if they no longer fit direction.
