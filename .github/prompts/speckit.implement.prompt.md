---
description: Execute all tasks from tasks.md to implement the feature according to the plan.
---

You are implementing a feature according to its specification, plan, and task list.

## Prerequisites

Required files (ERROR if missing):
- `specs/[###-feature]/spec.md`
- `specs/[###-feature]/plan.md`
- `specs/[###-feature]/tasks.md`

## Outline

### 1. Load context

Read ALL of the following before writing any code:
- `specs/[###-feature]/spec.md` — acceptance criteria are the definition of done
- `specs/[###-feature]/plan.md` — architecture, file structure, tech stack
- `specs/[###-feature]/tasks.md` — ordered task list
- `.specify/memory/constitution.md` — non-negotiable principles
- `specs/[###-feature]/data-model.md` (if present)
- `specs/[###-feature]/contracts/` (if present)
- `specs/[###-feature]/quickstart.md` (if present)

### 2. Execute tasks in order

Follow the phase structure from tasks.md:

1. **Phase 1 — Setup**: Initialize project structure.
2. **Phase 2 — Foundational**: Build blocking infrastructure. Verify before proceeding.
3. **Phase 3+ — User Stories**: Implement in priority order (P1 first).
   - After each story: verify acceptance scenarios from spec.md independently.
   - Only proceed to next story after current story is independently testable.
4. **Final Phase — Polish**: Apply cross-cutting improvements.

For each task:
- Mark it complete in `tasks.md`: change `[ ]` to `[x]`
- Parallel tasks `[P]` can be implemented simultaneously
- If tests are included: implement test first, verify it FAILS, then implement the code

### 3. Checkpoints

Stop and verify at each checkpoint defined in tasks.md:
- Run the quickstart validation from `specs/[###-feature]/quickstart.md`
- Confirm each acceptance scenario from the relevant user story in spec.md
- Do not proceed if a checkpoint fails — fix before moving on

### 4. Report completion

When all tasks are complete, output:
- Summary of what was implemented
- Which user stories are now independently functional
- Any deviations from the plan (with justification)
- Remaining TODOs or known limitations
- Suggested commit message

## Rules

- Acceptance criteria in spec.md override all other guidance.
- Never implement features not in the spec — create a new spec if needed.
- Every implemented user story must be independently testable before moving to the next.
- Constitution principles are non-negotiable — document any justified deviations.
