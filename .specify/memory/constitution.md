# DEX Swap Aggregator Constitution

**Version**: 1.2.0 | **Ratified**: 2026-04-02 | **Last Amended**: 2026-04-02

## Business Goal

Generate **€5,000+/month (~2,000,000 HUF) passive income** by charging a small
protocol fee (≤0.1%) on swap volume routed through the aggregator.  
Target: $360,000+/day in swap volume at 0.05% fee rate.  
Solo developer — simplicity and maintainability are survival requirements.

## Core Principles

### I. On-Chain Safety First (NON-NEGOTIABLE)

Smart contracts handle real user funds. Any vulnerability can cause total, irreversible
loss of user assets and destroy the project.

- All smart contract code MUST be covered by fork-based integration tests before deployment.
- All contracts MUST be audited (or at minimum peer-reviewed) before mainnet launch.
- No upgradeable proxy patterns unless explicitly justified — prefer immutable contracts.
- Emergency pause mechanisms MUST exist for all fund-handling contracts.
- Private keys and admin roles MUST use a hardware wallet or multisig; never a hot wallet.

### II. Revenue-Driven Simplicity (YAGNI)

Every feature must have a clear, measurable contribution to swap volume, user retention,
or operational cost reduction. Features without business justification MUST NOT be built.

- Start with the simplest working implementation; add complexity only when usage data proves the need.
- Prefer existing audited protocols and libraries over custom implementations.
- A feature is worth building only if it demonstrably increases swap volume or reduces churn.

### III. L2-First, EVM-Compatible Expansion

Start on a single high-volume EVM L2 (Base or Arbitrum One) to maximize user activity
per gas dollar spent. Expand to additional chains only after the core product is proven.

- Primary target: **Base** (high volume, low fees, Coinbase distribution, growing DeFi ecosystem).
- Secondary: Arbitrum One, Optimism — share the same codebase via chain config.
- Ethereum mainnet: supported in routing but not a launch priority.
- Non-EVM chains (Solana, etc.): out of scope until $1M+/month revenue is sustained.
- All chain-specific logic MUST be isolated behind a chain-config abstraction.

### IV. API-First Routing Engine

The quote/routing API is the core product and the primary integration surface.
It MUST be defined as an explicit contract before implementation begins.

- The routing engine API contract (request/response schema, error codes) MUST be finalized
  before any implementation code is written.
- Frontend and smart contracts are consumers of this API — they MUST not bypass it.
- Breaking API changes require a version bump and a migration guide.
- All API responses MUST include: best route, price impact, estimated gas, protocol fee breakdown.

### V. Observability & Revenue Monitoring

Silent failures in a revenue-generating system are unacceptable.
Every swap that fails, slips, or routes incorrectly is lost revenue and lost user trust.

- All swap events MUST be indexed on-chain and mirrored off-chain for revenue tracking.
- The system MUST emit alerts if swap success rate drops below 95% in any 1-hour window.
- Revenue dashboard (daily/monthly fee income by chain) MUST be operational from day one of mainnet.
- All off-chain services MUST expose a `/health` endpoint and structured logs.

## Constraints & Standards

- **EVM chains**: Solidity ^0.8.24, Foundry for contract development and testing.
- **Backend / Routing Engine**: Python 3.12+, FastAPI; blockchain interaction via web3.py 7+.
- **Frontend**: React 19 + Vite; wallet connection via wagmi v2 + viem v2 (JS/TS — browser wallet integration requires JavaScript).
- **No GPL dependencies** — all dependencies must be MIT, Apache 2.0, or BSL compatible.
- **No custodial patterns**: the aggregator contract MUST NOT hold user funds between transactions.
- **Slippage protection**: every swap MUST enforce a minimum output amount on-chain.
- **Hosting — Routing Engine**: Railway (managed PaaS); git push deploys automatically;
  always-on (no sleep/spin-down); ~$5/month. No server provisioning or OS maintenance required.
- **Hosting — Frontend**: Cloudflare Pages (free tier); git push deploys automatically;
  global CDN; no infrastructure management required.
- **Smart contracts**: deployed on-chain (Base); no hosting required after deployment.
- **No single third-party RPC**: routing engine MUST configure a fallback RPC provider
  (Alchemy primary, Ankr public fallback) to avoid a single point of failure.
- **Secret management**: no secrets in source code or git history; use environment variables + vault.

## Development Workflow

- **Solo developer** — no PR review required, but every spec MUST be written before implementation.
- **Branch strategy**: feature branches off `main` following `[###-feature-name]` naming.
- **Definition of Done** for each feature:
  - [ ] Spec accepted (spec.md complete, no open NEEDS CLARIFICATION)
  - [ ] Plan accepted (plan.md complete, Constitution Check passed)
  - [ ] All tasks completed (tasks.md all checked)
  - [ ] Fork tests pass on target chain
  - [ ] Revenue monitoring covers the new flow
  - [ ] Deployed to testnet and manually validated via quickstart.md
- **Mainnet deployment**: only after testnet validation + at least one independent code review.
- **No speculative features**: only build what is in a completed, accepted spec.

## Governance

This constitution supersedes all other development guidelines.  
Amendments require:

1. Written rationale documenting the reason for change
2. Review and approval by project maintainers
3. Backwards compatibility assessment for existing specs
4. Version bump per semantic versioning (MAJOR / MINOR / PATCH)

All feature specs and plans must comply with this constitution before
implementation begins. Constitution Check gates in `plan.md` enforce this.
