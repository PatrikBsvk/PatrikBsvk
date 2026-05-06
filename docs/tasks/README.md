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

## Milestones

| ID | Name | Goal | Tasks |
|---|---|---|---|
| **M1** | Foundation | Repo, Next.js, Supabase, auth, agent runner, marketing landing | T001–T006 |
| **M2** | Wizard + Builder Agent | Templates, wizard UI, Builder Agent, project workspace | T007–T013 |
| **M3** | Click-to-edit | WebContainers preview, UI Editor, Variant agent, auto-save | T014–T020 |
| **M4** | Deploy & subdomains | Vercel API integration, generated-app DB strategy (ADR-002) | T021–T025 |
| **M5** | Credits, billing, marketplace | Stripe subscriptions, kreditní counter, public profiles, Marketplace listings | T026–T035 |
| **M6** | Polish & soft launch | Coach interactions, smart errors, mobile preview, CI, Sentry, PostHog, beta launch | T036–T045 |

## Active queue (Ready)

- **T001** — Init monorepo
- **T002** — Supabase setup (after manual project creation)
- **T003** — DB schema + RLS + seed
- **T004** — App shell + auth + brand tokens
- **T005** — Agent runner + Coach
- **T006** — Marketing landing page
- **T007** — Wizard UI shell (M2)

## Coming next (PM is drafting)

- **T008** — Templates package + Landing template scaffold
- **T009** — Builder Agent (wires Wizard payload → file map)
- **T010** — Project workspace shell (3-column editor)
- **T011** — WebContainers live preview
- **T012** — UI Editor agent + click-to-edit
- **T013** — Variant agent (3-options picker)
- T014+ — see milestones above

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
