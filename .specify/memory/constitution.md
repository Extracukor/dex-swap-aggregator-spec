# [PROJECT_NAME] Constitution
<!-- Replace [PROJECT_NAME] with your project name, e.g. "DEX Swap Aggregator" -->

**Version**: 0.1.0 | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [RATIFICATION_DATE]

## Core Principles

### I. [PRINCIPLE_1_NAME]
<!-- Example: "Simplicity First" -->
[PRINCIPLE_1_DESCRIPTION]
<!-- Example: "Start with the simplest working solution. Add complexity only when
proven necessary by real requirements. YAGNI: You Aren't Gonna Need It." -->

### II. [PRINCIPLE_2_NAME]
<!-- Example: "User-Centric Design" -->
[PRINCIPLE_2_DESCRIPTION]
<!-- Example: "Every feature must serve a clear user need. Business value must be
demonstrable before implementation begins." -->

### III. [PRINCIPLE_3_NAME]
<!-- Example: "Test-First (NON-NEGOTIABLE)" -->
[PRINCIPLE_3_DESCRIPTION]
<!-- Example: "All implementation MUST follow TDD. Tests are written, reviewed, and
confirmed to fail (Red) before any implementation code is written." -->

### IV. [PRINCIPLE_4_NAME]
<!-- Example: "API-First Contracts" -->
[PRINCIPLE_4_DESCRIPTION]
<!-- Example: "All service interfaces must be defined as explicit contracts before
implementation. Internal teams are treated as API consumers." -->

### V. [PRINCIPLE_5_NAME]
<!-- Example: "Observability" -->
[PRINCIPLE_5_DESCRIPTION]
<!-- Example: "All features must emit structured logs and metrics from day one.
Debugging must be possible without attaching a debugger." -->

## Constraints & Standards

<!-- Document your non-negotiable technical constraints, compliance requirements,
and organizational standards here. Examples:
- Cloud provider: [e.g., Azure only]
- Supported languages/runtimes: [e.g., TypeScript 5+, Node.js 20 LTS]
- Authentication: [e.g., OAuth 2.0 with Azure AD]
- Data residency: [e.g., EU-only storage]
- Licensing: [e.g., no GPL dependencies]
-->

[CONSTRAINTS_CONTENT]

## Development Workflow

<!-- Document how the team works. Examples:
- Branch strategy: [e.g., feature branches, trunk-based]
- PR review: [e.g., 1 required approver]
- Definition of Done: [checklist items]
- Deployment cadence: [e.g., continuous delivery]
-->

[WORKFLOW_CONTENT]

## Governance

This constitution supersedes all other development guidelines.  
Amendments require:

1. Written rationale documenting the reason for change
2. Review and approval by project maintainers
3. Backwards compatibility assessment for existing specs
4. Version bump per semantic versioning (MAJOR / MINOR / PATCH)

All feature specs and plans must comply with this constitution before
implementation begins. Constitution Check gates in `plan.md` enforce this.
