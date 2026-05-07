# Vibell — Backlog

## How tasks work

Each task is a self-contained `.md` file with everything a coding agent needs:
- Title and ID (T001, T002, …)
- Milestone (M1–M6)
- Status (Ready · In Progress · Blocked · Done)
- Dependencies on other tasks
- Context (why this matters)
- Goal (the outcome)
- Acceptance criteria (testable)
- Files to create / modify
- Implementation notes (gotchas, tips)
- **Agent Prompt** — copy-paste this into your local Claude Code
- Definition of Done

## Phase 1 scope (2026-05-06 v0.4 — founder direction)

**Vibell = guided builder for non-programmers with a vision.** Primary user is anyone non-technical with an idea who wants to build an app or website. Vibell **enables, helps, and guides them to success.** Audience is broad (small entrepreneurs, creators, freelancers, students, hobbyists, indie hackers); output is narrow (web / one-pager / simple app).

Phase 1 ships the Studio product end-to-end — wizard, builder, workspace, click-to-edit, Studio Mode, mockup library, deploy — with 5 universal templates: Landing, Multi-page site, Portfolio, Booking, Shop one-pager.

Marketplace, Creator Hub, memberships, referral, GitHub sync, multi-user — all DEFERRED to Phase 2+.

## Milestones (Phase 1)

| ID | Name | Goal | Tasks |
|---|---|---|---|
| **M1** | Foundation | Repo, Next.js, Supabase, auth, agent runner, marketing landing | T001–T006 |
| **M2** | Wizard + Builder | Templates, wizard UI, Builder Agent, project workspace shell | T007–T010 |
| **M3** | Click-to-edit | WebContainers preview, UI Editor agent, Variant agent, auto-save | T011–T016 |
| **M4** | Deploy + SMB enablers | Vercel deploy, custom subdomains, AI Brand Designer, Smart Autofill, AI Debugger, built-in analytics, image gen | T017–T028 |
| **M5** | Credits, billing, soft launch | Stripe subscriptions, kreditní counter, smart errors, polish, 50-user beta with SMBs | T029–T036 |

## Milestones (Phase 2 — DEFERRED, designed later)
- Marketplace + listings
- Creator Hub (`/@username`, articles, tips, paid membership)
- Referral / Ambassador program
- Pre-built integrations library
- AI SEO Assistant
- Voice Mode
- Continuity Coach
- GitHub sync (Pro+)

## Active queue (Ready)

- **T001** — Init monorepo
- **T002** — Supabase setup (after manual project creation)
- **T003** — DB schema + RLS + seed
- **T004** — App shell + auth + brand tokens
- **T005** — Agent runner + Coach
- **T006** — Marketing landing page
- **T007** — Wizard UI shell (M2)
- **T008** — Templates package + Landing template scaffold
- **T009** — Builder Agent (memory injection, SSE progress, retry)
- **T010** — Workspace shell (Smart/Studio toggle, view switcher)

## Coming next (PM is drafting)

- **T011** — WebContainers live preview integration
- **T012** — UI Editor agent + click-to-edit interaction
- **T013** — Variant agent (3-options picker)
- **T014** — Auto-save snapshots + version restore UI
- **T015** — Memory Agent (background digest + post-wizard write upgrade)
- **T016** — Memory settings page (`/app/settings/memory`)
- **T017** — Local Business template (template #2)
- **T018** — Service Business template (template #3)
- **T019** — Booking template (template #4)
- **T020** — Shop one-pager template (template #5)
- T021+ — Vercel deploy, AI Brand Designer, Smart Autofill, AI Debugger, analytics, image gen

## Status legend
- **Ready** — fully specified, can be picked up
- **In Progress** — agent is working on it
- **Blocked** — waiting on a decision or another task
- **Done** — acceptance criteria met, committed, pushed

## Workflow for the founder

1. Pick the next "Ready" task with the lowest ID inside the active milestone.
2. Open the file. Read context.
3. Copy the **Agent Prompt** section.
4. Start a Claude Code session in your local repo, paste the prompt.
5. After the agent finishes, verify acceptance criteria.
6. Update the task `Status:` to **Done**.
7. Commit: `task(Txxx): <short title>`.
8. Move to next task.

## Workflow for the PM (Claude in this session)

When the founder asks for next tasks or for feature changes:
1. Update or add task files with full specs.
2. Keep `docs/tasks/README.md` index updated when a milestone is finished.
3. Re-evaluate dependencies and reorder if needed.
4. Add ADRs in `docs/architecture/adrs/` for any non-trivial decisions.
