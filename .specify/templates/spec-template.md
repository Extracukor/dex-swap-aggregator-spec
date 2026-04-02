# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]
**Status**: Draft
**Input**: User description: "[original description]"

---

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE — meaning if you implement
  just ONE of them, you still have a viable MVP that delivers value.

  Assign priorities (P1, P2, P3, ...) where P1 is the most critical.
-->

### User Story 1 — [Brief Title] (Priority: P1)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 2 — [Brief Title] (Priority: P2)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### Edge Cases

- What happens when [boundary condition]?
- How does the system handle [error scenario]?

---

## Requirements *(mandatory)*

<!--
  Focus on WHAT the system must do, not HOW.
  Use [NEEDS CLARIFICATION: specific question] for anything ambiguous.
-->

### Functional Requirements

- **FR-001**: System MUST [specific capability]
- **FR-002**: System MUST [specific capability]
- **FR-003**: Users MUST be able to [key interaction]

### Key Entities *(include if feature involves data)*

- **[Entity 1]**: [What it represents, key attributes — no implementation details]
- **[Entity 2]**: [What it represents, relationships to other entities]

---

## Success Criteria *(mandatory)*

<!--
  Must be measurable, technology-agnostic, and user-focused.
  Good: "Users can complete X in under 2 minutes"
  Bad: "API responds in 200ms" (implementation detail)
-->

### Measurable Outcomes

- **SC-001**: [Measurable metric, e.g., "Users can complete account creation in under 2 minutes"]
- **SC-002**: [Measurable metric, e.g., "System handles 1,000 concurrent users without degradation"]
- **SC-003**: [User satisfaction metric, e.g., "90% of users complete primary task on first attempt"]

---

## Assumptions

- [Assumption about target users, e.g., "Users have stable internet connectivity"]
- [Assumption about scope, e.g., "Mobile support is out of scope for v1"]
- [Dependency on existing system, e.g., "Requires access to existing user profile API"]
