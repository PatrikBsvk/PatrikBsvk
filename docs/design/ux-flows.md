# Vibell — UX Master Flows

**Status:** v0.1
**Purpose:** Single source of truth for end-to-end user journeys. Every product task references this.

---

## A. The "Wow" path — first-time visitor publishes their first app

**Goal:** From landing on vibell.app to a live URL in **under 5 minutes** (real interaction time, not counting Builder generation wait).

```
                            vibell.app (dark hero)
                                    │
                                    ▼  "Try Vibell free"
                            ┌───────────────────┐
                            │ Sign-up screen    │
                            │ email / Google /  │
                            │ GitHub            │
                            └─────────┬─────────┘
                                      ▼
                        ┌───────────────────────────┐
                        │  Welcome ("What do you    │
                        │   want to build?")        │
                        │                           │
                        │  [Pick a template]        │
                        │  [Start blank]            │
                        └─────────┬─────────────────┘
                                  ▼
                        ┌───────────────────────────┐
                        │ Wizard Step 1 — Template  │
                        │ 20 cards + "Start blank"  │
                        └─────────┬─────────────────┘
                                  ▼
                        ┌───────────────────────────┐
                        │ Wizard Step 2 — About it  │
                        │ Name, 1-sentence, audience│
                        │ + 3-5 feature chips       │
                        │ (voice input optional)    │
                        └─────────┬─────────────────┘
                                  ▼
                        ┌───────────────────────────┐
                        │ Wizard Step 3 — Mockup    │
                        │ 3 layout variants visual  │
                        └─────────┬─────────────────┘
                                  ▼
                        ┌───────────────────────────┐
                        │ Wizard Step 4 — Style     │
                        │ Minimal / Bold / Playful  │
                        │ → "Build my app" CTA      │
                        └─────────┬─────────────────┘
                                  ▼
                       ┌─────────────────────────────┐
                       │  Builder Agent generating   │
                       │  (animated, ~30 s, hides    │
                       │   token spend behind ETA)   │
                       └─────────┬───────────────────┘
                                 ▼
                       ┌─────────────────────────────┐
                       │  Workspace opens            │
                       │  ─ Sidebar (Pages/Style)    │
                       │  ─ Live preview (iframe)    │
                       │  ─ Coach panel              │
                       │                             │
                       │  Click any element → edit   │
                       └─────────┬───────────────────┘
                                 ▼  "Publish"
                       ┌─────────────────────────────┐
                       │  Pick subdomain             │
                       │  mysushi.vibell.app         │
                       └─────────┬───────────────────┘
                                 ▼
                       ┌─────────────────────────────┐
                       │  Live URL                   │
                       │  "Share your work?"         │
                       │  [Add to portfolio]         │
                       │  [List on Marketplace]      │
                       └─────────────────────────────┘
```

### Time budget
| Step | Target time |
|---|---|
| Sign-up | 30 s |
| Welcome → Wizard step 1 | 5 s |
| Wizard step 1 (template) | 20 s |
| Wizard step 2 (about) | 60 s |
| Wizard step 3 (mockup) | 15 s |
| Wizard step 4 (style) | 10 s |
| Builder generates | 30 s (visible), real ≤ 60 s |
| Workspace explore | 60 s |
| Publish | 10 s |
| **Total interactive** | **~3 minutes** |

### Hand-offs between flows
- After publish → "Share your work?" is the entry to the **Social Network** flow (D below).
- After workspace explore (no publish) → returns to Dashboard with project listed.

---

## B. Returning user — editing an existing app

```
Sign in → Dashboard → Project card → Workspace → edit → auto-save → publish
```

Key states:
- **Project status:** Draft | Live | Archived (badge in dashboard card)
- **Last edit:** "Edited 2 h ago" timestamp
- **Quick actions on hover:** Open · Duplicate · Archive · Share

---

## C. Visitor browsing the Marketplace (Social Network pillar)

```
1. Land on vibell.app/store (no login required)
2. Browse grid of public apps with:
   - Live thumbnail (iframe screenshot or live preview on hover)
   - Title + creator handle (@username)
   - Free / paid badge with price
   - Category chips
3. Click → app detail page:
   - Full preview (interactive iframe)
   - Description + creator bio
   - Reviews + clones count
   - "Clone for ___ credits" or "Buy for $___"
4. Buy / Clone → if not signed in, sign-in modal first
5. After clone: copy lands in their workspace, ready to edit
```

Filters: category, price range, technology features, popularity.

---

## D. Creator listing their app on the Marketplace

```
From workspace top-bar → "List on Vibell Store"
   ↓
1. Listing form:
   - Title (pre-filled with project name)
   - Description (AI-suggest button: Coach drafts from project context)
   - Cover image (auto-generated screenshot, replaceable)
   - Category (selector)
   - Pricing model:
     a. Free clone (visibility + analytics for creator)
     b. Pay-once clone (creator sets USD price, min $5)
     c. Pay-monthly (template subscription, future feature)
   - Revenue split shown explicitly (70/30 free + basic, 80/20 Pro+)
2. Stripe Connect onboarding (first time): creator onboards an Express account
3. Submit → manual review queue (auto-approved for first 100 users to bootstrap)
4. Once live, "Manage" button shows: views, clones, revenue
```

---

## E. Buyer purchases / clones an app

```
On app detail page → "Clone for 9 credits" or "Buy for $19"
   ↓
- If credits action: deduct credits via charge_credits RPC, copy file map to new project
- If money action: Stripe Checkout, on completion:
  - Create new project under buyer's account
  - 70/30 (or 80/20) split: payout queued to creator's Stripe Connect
  - 30 (or 20) % to Vibell account
  - Email receipt to both sides
```

---

## F. Public profile / portfolio

`vibell.app/@username` shows:
- Avatar, display name, bio
- Stats: apps built, clones received, total earned (if creator opted to show)
- Grid of public apps
- Follow button

**Why this matters:** the portfolio doubles as a **social network entry** and a **lead gen for the creator** (they can put it on their CV / Twitter). Vibell becomes a creator's home base, not just a tool.

---

## G. Empty / failure / friction states

| State | What we show |
|---|---|
| 0 credits before action | Modal: "You need 5 credits. Upgrade to Basic, or buy a 100-pack for $8.50." |
| Builder fails | Friendly: "We couldn't finish that. Often a template change fixes it. Retry, or pick another template." (offer 1-click retry, or pick template) |
| Publish fails (Vercel down) | "Your app is saved. We'll publish as soon as our deploy service is back. We'll email you." |
| User abandons mid-wizard | Wizard state saved as draft. On return: "Continue where you left off?" |
| Slow Builder (>60 s) | After 30 s show: "Still working — large apps take a moment. You can stay or we'll email you when it's ready." |
| Cold-started free-tier app | "Waking up..." 5 s, then loads. (Auto-pause after 24 h idle for free.) |

---

## Cross-cutting principles
1. **Never block** with full-screen loaders. Always show progress with ETA.
2. **Always undoable.** Every action has either undo or a snapshot to revert to.
3. **Cost transparency.** Before any AI action, show credit cost.
4. **No emojis.** Brand voice rule.
5. **Coach is a panel, never a popup.** The user is in control of when to talk.
