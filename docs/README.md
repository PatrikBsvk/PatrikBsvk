# Vibell — Documentation

Single source of truth for the Vibell product. Files here are written so they can be fed directly to a coding agent (Claude Code Opus 4.7) as context.

## Structure

```
docs/
├── README.md                 (this file)
├── plan.md                   Top-level plan: vision, MVP scope, roadmap
├── product/
│   ├── vision.md             Product vision (one-liner, why now, success metrics)
│   ├── personas.md           Target users
│   ├── competition.md        Market & positioning vs. base44, Lovable, etc.
│   └── brand.md              Brand identity (WIP)
├── architecture/
│   ├── overview.md           System layers, repo layout
│   ├── agents.md             Agent system (Builder, UI Editor, ...)
│   └── credits.md            Credit-based pricing spec
├── design/                   (TBD — design system, UX flows)
└── tasks/
    ├── README.md             Backlog format & milestone index
    └── Mx-<name>/Txxx-*.md   Individual task files
```

## How to use with your local coding agent

1. Pick the next task from `docs/tasks/` (lowest "Ready" ID inside the active milestone).
2. Open it. The "Agent Prompt" section is self-contained — copy-paste into your local Claude Code.
3. The agent reads referenced docs (vision, architecture) as needed.
4. Verify acceptance criteria in the task file.
5. Update the task `Status:` to Done, commit with `task(Txxx): <title>`.

## Conventions

- Tasks are numbered sequentially (T001, T002, …) regardless of milestone.
- Milestones M1–M6 follow `docs/plan.md` MVP timeline.
- Every task has acceptance criteria + a self-contained agent prompt.
- Reference docs are linked from tasks; agent should read them before coding.
