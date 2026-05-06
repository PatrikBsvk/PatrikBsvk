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
| **M1** | Foundation | Repo, Next.js app, Supabase, auth, layout | T001–T010 |
| **M2** | Wizard + Builder Agent | Templates, wizard UI, first agent | T011–T020 |
| **M3** | Live preview + UI Editor | WebContainers, click-to-edit | T021–T030 |
| **M4** | Deploy & subdomains | Vercel API integration | T031–T035 |
| **M5** | Credits & Stripe | Billing, kreditní counter | T036–T042 |
| **M6** | Polish & soft launch | Coach, smart errors, mobile preview | T043–T050 |

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
