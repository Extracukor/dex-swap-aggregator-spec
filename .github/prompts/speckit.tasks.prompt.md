---
description: Generate an executable task list from the implementation plan and design artifacts.
---

You are generating a task list for implementing a specified and planned feature.

## Prerequisites

Required files (ERROR if missing):
- `specs/[###-feature]/spec.md` — user stories and requirements
- `specs/[###-feature]/plan.md` — implementation plan

Optional (use if present):
- `specs/[###-feature]/data-model.md`
- `specs/[###-feature]/contracts/`
- `specs/[###-feature]/research.md`

## Outline

### 1. Setup

- Identify the current feature folder: `specs/[###-*]/`
- Read all available files listed above.
- Copy `.specify/templates/tasks-template.md` to `specs/[###-feature]/tasks.md`.

### 2. Analyze inputs

Extract from spec.md:
- User stories with their priorities (P1, P2, P3, ...)
- Acceptance scenarios per story

Extract from plan.md:
- Technology stack and project structure
- Phase structure and dependencies

Extract from data-model.md (if present):
- Entities to be implemented

Extract from contracts/ (if present):
- Endpoints/interfaces to be implemented

### 3. Generate tasks

Organize tasks by phase:

- **Phase 1 — Setup**: Project initialization, dependencies, tooling. No story dependency.
- **Phase 2 — Foundational**: Core infrastructure that ALL user stories depend on.
  Mark as BLOCKING — no story work begins until this phase is complete.
- **Phase 3+ — User Stories**: One phase per user story, ordered by priority (P1 first).
  Each story's tasks must be independently completable and testable.
- **Final Phase — Polish**: Cross-cutting concerns (docs, refactoring, security hardening).

For each task:
- Assign an ID: T001, T002, ...
- Mark parallel tasks with `[P]` (different files, no dependencies)
- Tag story tasks with `[US1]`, `[US2]`, etc.
- Include exact file paths in descriptions (based on plan.md structure)
- If tests were requested: write test tasks FIRST, mark them to fail before implementation

### 4. Add dependency summary

At the end of tasks.md, document:
- Phase dependencies (what must complete before what)
- Parallel opportunities
- Implementation strategy (MVP-first, incremental delivery, or parallel team)

### 5. Report completion

Output:
- Tasks file path: `specs/[###-feature]/tasks.md`
- Total task count
- MVP scope (P1 user story tasks only)
- Next step: run `/speckit.implement` to start implementation

## Rules

- Every task must be atomic and verifiable.
- Each user story phase must produce an independently testable MVP increment.
- Tests (if included) MUST be written and confirmed to FAIL before implementation tasks.
- Avoid vague tasks like "implement the feature" — be specific about files and behavior.
