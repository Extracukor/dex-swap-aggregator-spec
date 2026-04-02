---
description: Create a technical implementation plan from an existing feature specification.
---

You are creating an implementation plan for a feature that has already been specified.

## User Input

The user may provide tech stack preferences and architectural choices.
If not provided, make informed recommendations based on the project constitution and spec.

## Outline

### 1. Setup

- Identify the current feature: find the most recently created or active `specs/[###-*]/` folder.
- Read `specs/[###-feature]/spec.md` (required).
- Read `.specify/memory/constitution.md` (required — all plans must comply).
- Copy `.specify/templates/plan-template.md` to `specs/[###-feature]/plan.md`.

### 2. Fill Technical Context

Fill in the Technical Context section of `plan.md`:
- Use user input for tech stack if provided; otherwise recommend based on constitution constraints.
- Mark any unknowns as `NEEDS CLARIFICATION`.

### 3. Constitution Check

Evaluate each principle from `.specify/memory/constitution.md`:
- For each principle, explicitly check if the proposed plan complies.
- **GATE**: If a violation exists and is not justified, ERROR and refuse to continue
  until the plan is revised or the violation is documented in Complexity Tracking.

### 4. Phase 0 — Research

For each `NEEDS CLARIFICATION` in Technical Context:
- Research the options and document a decision with rationale.
- Write findings to `specs/[###-feature]/research.md` using format:
  - **Decision**: what was chosen
  - **Rationale**: why this was chosen
  - **Alternatives Considered**: what else was evaluated

### 5. Phase 1 — Design & Contracts

Prerequisites: `research.md` complete.

1. Extract entities from the feature spec → write `specs/[###-feature]/data-model.md`
   - Entity name, fields, relationships, validation rules, state transitions.

2. Define interface contracts → write `specs/[###-feature]/contracts/`
   - Appropriate format for project type (REST endpoints, CLI schema, event contracts, etc.)
   - Skip if project is purely internal with no external interfaces.

3. Write `specs/[###-feature]/quickstart.md` with key validation scenarios
   (the minimum steps to prove the feature works end-to-end).

### 6. Report completion

Output:
- Plan file path: `specs/[###-feature]/plan.md`
- Generated artifacts: research.md, data-model.md, contracts/, quickstart.md
- Constitution compliance status
- Next step: run `/speckit.tasks` to generate the executable task list

## Key Rules

- The plan must remain high-level and readable.
- Any code samples or detailed algorithms belong in `implementation-details/` files.
- ERROR on gate failures or unresolved clarifications after research.
- Technical choices must trace back to spec requirements.
