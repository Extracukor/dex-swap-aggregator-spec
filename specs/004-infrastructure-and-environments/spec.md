# Feature Specification: Infrastructure and Environments

**Feature Branch**: `004-infrastructure-and-environments`
**Created**: 2026-04-05
**Status**: Draft
**Input**: User description: "Define the deployment pipeline and environment secrets for the project."

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Multi-Environment Deployments Work Reliably (Priority: P1)

As the project owner, I need clear Dev, Test (Staging), and Prod environments so changes can be validated safely before production release.

**Why this priority**: Environment isolation is the baseline for safe shipping. Without it, release risk is too high.

**Independent Test**: Trigger deployments from the expected branches and confirm artifacts, URLs, and secrets are isolated per environment.

**Acceptance Scenarios**:

1. **Given** code is pushed to `dev`, **When** the backend deployment pipeline runs, **Then** Railway deploys the service to the Dev environment only.
2. **Given** a pull request is opened for frontend changes, **When** CI builds the frontend, **Then** Cloudflare Pages creates a Preview deployment linked to the PR.
3. **Given** code is merged into `main`, **When** deployment workflows run, **Then** backend and frontend are deployed to Production targets only.
4. **Given** a release candidate must be validated, **When** Test (Staging) deployment is triggered, **Then** the release is available in an isolated staging environment with staging secrets.

---

### User Story 2 - Secrets and Wallet Signing Stay Secure (Priority: P1)

As the operator, I need a secure secret and wallet strategy so automation can sign transactions when needed without exposing credentials.

**Why this priority**: Wallet and API key leakage is catastrophic for user trust, funds, and uptime.

**Independent Test**: Run security checks and deployment validation to ensure all required secrets are resolved from Railway/Cloudflare secret stores, with no plaintext secrets in repository files or logs.

**Acceptance Scenarios**:

1. **Given** a bot signing workflow is executed, **When** it needs signing authority, **Then** it uses either CDP hosted wallets or encrypted environment variables per configured policy.
2. **Given** deployment starts in any environment, **When** runtime initializes, **Then** DEX, social, RPC, and wallet credentials are loaded only from platform secret managers.
3. **Given** application logs are reviewed, **When** signing and API integrations run, **Then** private keys and secrets are never present in logs.

---

### User Story 3 - Growth Integrations Run Automatically (Priority: P2)

As a growth operator, I need reliable integration with Twitter/X and Farcaster (Neynar API) so growth updates can be published automatically without manual posting.

**Why this priority**: Consistent distribution improves discoverability and supports pre-launch and post-launch growth loops.

**Independent Test**: Execute scheduled or event-driven publication jobs and verify successful delivery to Twitter/X and Neynar-backed Farcaster posting endpoints.

**Acceptance Scenarios**:

1. **Given** a scheduled growth job is triggered, **When** the publisher executes, **Then** it can post to Twitter/X using environment-specific credentials.
2. **Given** Farcaster publication is enabled, **When** the same job executes, **Then** it can post via Neynar API with appropriate authentication.
3. **Given** one social API is unavailable, **When** publication fails for that channel, **Then** failure is isolated, logged, and retried without stopping the rest of the pipeline.

---

### Edge Cases

- What happens if `main` is merged while Test (Staging) validation is still running? -> Production deployment must remain gated by explicit workflow conditions.
- What happens if a required secret is missing in one environment? -> Deployment fails fast with a clear error and no partial startup.
- What happens if social provider credentials are revoked? -> Publication jobs fail safely, emit alerts, and do not impact swap routing.
- What happens if Railway or Cloudflare deployment APIs are temporarily unavailable? -> The workflow retries with backoff and reports terminal failure.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST define and maintain three isolated environments: Dev, Test (Staging), and Prod.
- **FR-002**: The backend Node.js service MUST be hosted on Railway.
- **FR-003**: Backend continuous deployment MUST map GitHub branches as follows: `dev` -> Dev environment, `main` -> Prod environment.
- **FR-004**: The deployment process MUST support Test (Staging) releases in an isolated environment for pre-production validation.
- **FR-005**: The frontend React application MUST be hosted on Cloudflare Pages.
- **FR-006**: Cloudflare Pages MUST create Preview deployments for pull requests and Production deployments for merges to `main`.
- **FR-007**: Automated bot signing MUST use either Coinbase Developer Platform (CDP) hosted wallets or encrypted environment variables, according to environment policy.
- **FR-008**: The Growth Engine MUST integrate with Twitter API (X) for automated publication flows.
- **FR-009**: The Growth Engine MUST integrate with Neynar API for Farcaster publication flows.
- **FR-010**: All API keys and credentials (DEX, social, RPC, wallets) MUST be stored only in Railway or Cloudflare secret stores and MUST NEVER be committed to source code.
- **FR-011**: The runtime and CI/CD logs MUST redact or exclude secret values and private key material.
- **FR-012**: Secret loading failures MUST block deployment or job execution before any external calls are attempted.

### Key Entities

- **EnvironmentProfile**: Defines Dev, Test (Staging), and Prod settings including deployment targets, branch triggers, and secret namespaces.
- **DeploymentMapping**: Rule set mapping GitHub events and branches to Railway and Cloudflare deployment destinations.
- **SecretReference**: Named pointer to a secret value stored in Railway or Cloudflare secret managers (without storing plaintext in code).
- **WalletSignerProfile**: Signing strategy descriptor indicating CDP hosted wallet usage or encrypted environment variable usage per automation context.
- **SocialIntegrationConfig**: Environment-specific configuration for Twitter/X and Neynar API publishing.
- **PublicationJobRun**: Audit record for each automated publication execution, including channel-level outcome and retry status.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of backend deployments from `dev` land in Dev and 100% from `main` land in Prod over a 30-day period.
- **SC-002**: 100% of frontend pull requests produce a Cloudflare Preview deployment URL, and 100% of `main` merges produce a Production deployment.
- **SC-003**: Zero detected plaintext secrets in repository history and CI/CD logs after enforcement is enabled.
- **SC-004**: 95% or more of scheduled publication jobs complete successfully per channel (Twitter/X, Neynar) over a rolling 30-day window.
- **SC-005**: Missing or invalid secrets are detected before runtime side effects in 100% of failing deployment/job runs.

---

## Assumptions

- The repository branch strategy includes active `dev` and `main` branches.
- Test (Staging) can be triggered via workflow rules (manual dispatch and/or release branch policy).
- Railway and Cloudflare accounts support per-environment secret management.
- Twitter/X and Neynar API credentials are available before automation rollout.
- CDP hosted wallets are available for the targeted networks used by bot automation.
