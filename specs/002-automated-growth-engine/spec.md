# Feature Specification: Automated Growth Engine

**Feature Branch**: `002-automated-growth-engine`
**Created**: 2026-04-02
**Status**: Draft
**Input**: User description: "automatizált promóció — napi swap statisztikák publikus közzététele és listing DeFi tracker oldalakon, kézi beavatkozás nélkül"

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Daily Stats Published Automatically (Priority: P1)

Every day, without any manual action, the protocol's swap volume, fee income, and
swap count for the previous 24 hours is published to a public channel. A potential
user or integrator who sees the post can verify the numbers directly on-chain.

**Why this priority**: The fastest path to organic reach with zero ongoing time cost.
Each published post is a self-contained proof-of-work: real volume, real fees, verifiable
on-chain. This builds credibility before the protocol is widely known.

**Independent Test**: Wait 24 hours after at least one swap occurs, then check the
public channel — a stats post must be present with accurate figures.

**Acceptance Scenarios**:

1. **Given** at least one swap occurred in the last 24 hours,
   **When** the daily publication window passes (e.g. 06:00 UTC),
   **Then** a public post appears containing: total swap count, total volume (USD),
   total fee income (USD), and a link to verify on-chain.

2. **Given** zero swaps occurred in the last 24 hours,
   **When** the daily publication window passes,
   **Then** no post is published (no "0 swaps today" noise).

3. **Given** the publication channel is temporarily unavailable (API error),
   **When** the publication attempt fails,
   **Then** the system retries up to 3 times, logs a structured alert,
   and swap routing continues unaffected.

---

### User Story 2 — Protocol Listed on DeFi Tracker Sites (Priority: P1)

A DeFi user who searches for "Base DEX aggregators" on DeFiLlama finds the protocol
listed with live volume data. No manual submission is required after the initial
one-time integration effort.

**Why this priority**: DeFiLlama is the primary discovery surface for DeFi protocols.
Listing provides continuous organic traffic and credibility without ongoing effort.
Volume data updates automatically from on-chain events.

**Independent Test**: After mainnet launch, visit `defillama.com/dex-aggregators` and
confirm the protocol appears with non-zero volume.

**Acceptance Scenarios**:

1. **Given** the protocol is live on Base mainnet with at least one day of swap volume,
   **When** a user visits DeFiLlama's DEX Aggregators section,
   **Then** the protocol appears with accurate daily and total volume figures.

2. **Given** new swaps occur on-chain,
   **When** DeFiLlama refreshes its data (within 24 hours),
   **Then** the displayed volume updates to reflect the new swaps without any manual action.

---

### User Story 3 — Public On-Chain Analytics Dashboard (Priority: P2)

A developer or investor who wants to verify the protocol's performance can open a
public analytics dashboard and see historical swap volume, fee income, and route
distribution — all derived directly from on-chain data.

**Why this priority**: A public dashboard lets the community audit and share
performance data independently, increasing trust and reducing inbound "what's your volume?"
questions. Lower priority than P1/P2 because it requires the P1 stories to have
data to display first.

**Independent Test**: Open the public dashboard URL, verify it shows historical data
matching on-chain records for at least the last 7 days.

**Acceptance Scenarios**:

1. **Given** the dashboard is published,
   **When** a visitor opens it,
   **Then** they see: daily volume chart (last 30 days), cumulative fee income,
   swap count over time, and DEX route distribution (% Uniswap v3 vs Aerodrome).

2. **Given** a user wants to verify a specific day's data,
   **When** they hover/click a data point,
   **Then** they can see a block explorer link to verify the underlying on-chain events.

---

### Edge Cases

- What if the on-chain RPC is unavailable at stats collection time? → Retry up to
  3 times with exponential backoff; if all retries fail, skip publication for that day
  and emit a `CRITICAL` log. Do not publish incomplete or estimated figures.
- What if the Twitter/X API account is rate-limited or suspended? → Log a `CRITICAL`
  alert, skip publication for that window. Swap routing is completely unaffected.
- What if the DeFiLlama adapter PR is rejected or delayed? → CoinGecko DEX section
  serves as the fallback listing target; both share the same data format.
- What if a published stat contains an error? → No automated correction; operator
  manually posts a correction tweet. Accuracy > speed.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST automatically publish daily swap statistics (swap count,
  total volume in USD, total fee income in USD) to at least one public channel without
  any manual trigger.
- **FR-002**: All published statistics MUST be derived exclusively from verified
  on-chain swap events — no estimation, approximation, or off-chain data sources.
- **FR-003**: Published figures MUST be accurate to the precision displayed; no
  rounding that would misrepresent the true on-chain value by more than 0.1%.
- **FR-004**: The publication system MUST NOT publish when zero swaps occurred in
  the measurement window — silence is preferable to zero-value noise.
- **FR-005**: A publication failure (network error, API rejection, rate limit) MUST
  NOT affect swap routing availability. The two systems MUST be fully isolated.
- **FR-006**: The publication system MUST retry failed attempts at least 3 times
  before marking a window as skipped.
- **FR-007**: All credentials for publication channels (API keys, tokens) MUST be
  stored in secret management and never appear in source code or logs.
- **FR-008**: The protocol MUST be listed on DeFiLlama's DEX Aggregators section within
  4 weeks of mainnet launch, with volume data updating automatically from on-chain events.
- **FR-009**: Each published post MUST include a verifiable link (block explorer or
  public dashboard) so any reader can independently confirm the figures.
- **FR-010**: The system MUST emit a structured alert when a publication window is
  skipped due to repeated failures.

### Key Entities

- **DailyStats**: Aggregated record for a 24-hour UTC window — swap count, volume (USD),
  fee income (USD), top token pairs, DEX route distribution. Derived from `SwapExecuted`
  events.
- **PublicationRecord**: Log entry for each publication attempt — window date, channel,
  success/failure, retry count, published content hash.
- **ChannelConfig**: Configuration for each publication channel — type (social post,
  analytics tracker), credentials reference (secret name), publication schedule.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Daily stats appear in the public channel within 2 hours of the 06:00 UTC
  publication window, every day that at least one swap occurred — with zero manual actions.
- **SC-002**: Published volume and fee figures match the on-chain `SwapExecuted` event
  totals with 100% accuracy (no rounding discrepancy > 0.1%).
- **SC-003**: A publication failure does not cause any degradation in swap quote or
  execution availability (independently verifiable via `/v1/health`).
- **SC-004**: The protocol appears on DeFiLlama's DEX Aggregators list within 4 weeks
  of mainnet launch.
- **SC-005**: The public analytics dashboard is available at a stable URL and requires
  zero recurring maintenance after initial setup.
- **SC-006**: Any reader of a published stats post can independently verify the figures
  within 3 clicks (post → block explorer or dashboard → on-chain data).

---

## Assumptions

- Twitter/X Developer account (Basic tier) will be obtained before mainnet launch;
  the ~$100/month API cost is justified once daily volume exceeds $20k.
- DeFiLlama accepts adapter PRs for protocols with verified on-chain volume;
  the one-time PR submission effort (~2 hours) is not counted as "ongoing maintenance."
- v1 scope: Twitter/X + DeFiLlama + one public analytics dashboard (Dune Analytics).
  Additional channels (Farcaster, Discord bot, CoinGecko) are future scope.
- The publication system runs on the same Railway infrastructure as the routing engine —
  no additional hosting cost.
- Published stats are read-only aggregates; no user PII is collected or published.
- USD conversion for volume figures uses the same Chainlink ETH/USD price feed
  already used by the routing engine (no new price oracle dependency).
- The analytics dashboard (Dune) is a one-time query setup; Dune refreshes the
  underlying data automatically from on-chain events.
