---
description: Create a new feature specification from a natural language description.
---

You are creating a new feature specification for this project.

## User Input

The text the user typed after invoking this prompt **is** the feature description.

## Outline

Given the feature description, do the following:

### 1. Determine the feature number and branch name

- Scan the `specs/` directory to find existing feature folders (e.g., `001-*`, `002-*`).
- The next feature number is the highest existing number + 1 (zero-padded to 3 digits).
- Generate a concise 2–4 word branch name from the description using action-noun format
  (e.g., "add-user-auth", "swap-route-optimizer").
- The feature folder will be: `specs/[###-branch-name]/`

### 2. Create the feature directory and spec file

- Create the directory: `specs/[###-branch-name]/`
- Copy `.specify/templates/spec-template.md` to `specs/[###-branch-name]/spec.md`
- Fill in the metadata header: feature name, branch, today's date.

### 3. Fill the specification

Follow this execution flow:

1. Parse the user description and extract: actors, actions, data, constraints.
2. For unclear aspects:
   - Make informed guesses based on context and industry standards.
   - Use `[NEEDS CLARIFICATION: specific question]` ONLY for critical decisions where
     multiple reasonable interpretations exist with different implications.
   - **Maximum 3 NEEDS CLARIFICATION markers total.**
3. Fill **User Scenarios & Testing** — at least 2 independent user stories with priorities.
4. Fill **Requirements** — measurable functional requirements (FR-001, FR-002, ...).
5. Fill **Success Criteria** — technology-agnostic, measurable outcomes (SC-001, ...).
6. Fill **Assumptions** — document reasonable defaults chosen when spec was vague.

### 4. Quality validation

After writing the spec, validate it:

- [ ] No implementation details (no tech stack, framework names, API specifics)
- [ ] Focused on user value and business need
- [ ] All mandatory sections completed
- [ ] Requirements are testable and unambiguous
- [ ] Success criteria are measurable and technology-agnostic
- [ ] Acceptance scenarios are defined for every user story
- [ ] Maximum 3 NEEDS CLARIFICATION markers remain

If items fail: update the spec to address them, then re-validate (max 3 iterations).

If NEEDS CLARIFICATION markers remain: present each as a structured question with 3
suggested answers and implications, wait for the user's response, then update the spec.

### 5. Report completion

Output:
- Feature branch name: `[###-branch-name]`
- Spec file path: `specs/[###-branch-name]/spec.md`
- Validation checklist summary
- Next step: run `/speckit.plan` to create the implementation plan

## Quick Guidelines

- Focus on **WHAT** users need and **WHY** — not HOW to implement.
- Written for business stakeholders, not developers.
- Every user story must be independently testable and deliverable as an MVP increment.
- Good success criteria: "Users can complete checkout in under 3 minutes"
- Bad success criteria: "API responds in 200ms" (implementation detail)
