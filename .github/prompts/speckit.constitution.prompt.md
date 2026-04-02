---
description: Create or update the project constitution — governing principles and development guidelines.
---

You are updating the project constitution at `.specify/memory/constitution.md`.

## User Input

The user may provide specific principles, constraints, or amendments.
If no input is provided, guide them through creating the constitution interactively.

## Outline

Follow this execution flow:

1. **Load** the existing `.specify/memory/constitution.md`.
   - Identify every placeholder token of the form `[ALL_CAPS_IDENTIFIER]`.
   - Note which sections still contain template placeholders.

2. **Collect values for placeholders**:
   - Use user input from this conversation if provided.
   - Infer from existing repo context (README, docs, existing specs).
   - For governance dates: today's date is the ratification date if first creation.
   - `CONSTITUTION_VERSION` follows semantic versioning:
     - MAJOR: Removal or redefinition of core principles
     - MINOR: New principle or section added
     - PATCH: Clarifications, wording, typo fixes

3. **Draft the updated constitution**:
   - Replace every placeholder with concrete text.
   - Ensure each Principle section has: succinct name, clear non-negotiable rules, rationale.
   - Governance section must include amendment procedure and compliance expectations.
   - No bracketed tokens left without explicit justification.

4. **Write** the completed constitution back to `.specify/memory/constitution.md`.

5. **Output a summary** including:
   - Version and bump rationale
   - Principles defined or changed
   - Suggested commit message

## Formatting Requirements

- Use Markdown headings exactly as in the template (do not change heading levels).
- Dates in ISO format: YYYY-MM-DD.
- Principles must be declarative and testable — replace "should" with MUST/SHOULD.
- Keep a single blank line between sections.
