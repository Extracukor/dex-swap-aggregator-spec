# Feature Specification: Core Swap Aggregation — Quote & Execute

**Feature Branch**: `001-core-swap-aggregation`
**Created**: 2026-04-02
**Status**: Draft
**Input**: User description: "A felhasználó megad egy token párt (pl. ETH → USDC) és egy
összeget, a rendszer megmutatja a legjobb swap árat több DEX-ről összehasonlítva, és végre
tudja hajtani a swapot Base-en a lehető legjobb route-on, a protokoll kis díjat von le."

---

## User Scenarios & Testing *(mandatory)*

<!--
  This is the MVP feature — the entire business model depends on it.
  Each user story is independently testable and delivers standalone value.
-->

### User Story 1 — Get Best Swap Quote (Priority: P1)

A trader wants to swap tokens on Base. They provide a token pair (e.g. ETH → USDC)
and an input amount. The system queries multiple DEX sources simultaneously, compares
the output amounts, and presents the best available price — including price impact,
estimated gas cost, and the protocol fee that will be deducted.

**Why this priority**: Without a quote, no swap can happen. This is the revenue-generating
entry point. Users who see a better price here will execute a swap; users who don't will leave.

**Independent Test**: A user can enter a token pair and amount, receive a quote with
best output amount, price impact, gas estimate, and fee breakdown — without connecting
a wallet or executing any transaction.

**Acceptance Scenarios**:

1. **Given** a user inputs ETH → USDC with amount 0.1 ETH,
   **When** they request a quote,
   **Then** the system returns the best output amount aggregated from at least 2 DEX
   sources on Base, with price impact %, estimated gas cost, and protocol fee amount
   — all within 2 seconds.

2. **Given** a user requests a quote,
   **When** the system queries DEX sources,
   **Then** the response clearly indicates which DEX provides the best rate and the
   output amount difference compared to the second-best source.

3. **Given** a user inputs a very large amount that would cause high price impact (>5%),
   **When** the quote is returned,
   **Then** the system MUST display a prominent high-price-impact warning.

4. **Given** a DEX source is temporarily unavailable,
   **When** a quote is requested,
   **Then** the system still returns a valid quote from the remaining available sources
   without failing entirely.

---

### User Story 2 — Execute Swap on Best Route (Priority: P1)

A trader has reviewed the quote and wants to execute the swap. They confirm the
transaction. The system routes the swap through the best DEX, deducts the protocol
fee on-chain, and delivers the output tokens to the user's wallet. The swap is atomic:
either it completes fully or it reverts — no partial fills, no stuck funds.

**Why this priority**: Execution is the revenue event. Every completed swap generates
protocol fee income. This story is inseparable from the quote in terms of user value
and must ship alongside it.

**Independent Test**: A user can connect a wallet on Base testnet, get a quote, approve
the input token (if ERC-20), and execute a swap that results in: output tokens received,
protocol fee deducted and recorded on-chain, transaction link provided.

**Acceptance Scenarios**:

1. **Given** a user has a valid quote and sufficient wallet balance,
   **When** they confirm the swap,
   **Then** the input tokens are transferred, the swap executes on the best route,
   the protocol fee is deducted, and the remaining output tokens land in their wallet —
   all within a single transaction.

2. **Given** the market moves and the actual output falls below the user's slippage
   tolerance (default 0.5%),
   **When** the transaction is submitted on-chain,
   **Then** the smart contract MUST revert the transaction and the user loses only gas,
   not their input tokens.

3. **Given** a swap completes successfully,
   **When** the transaction is confirmed,
   **Then** the system records the swap event (tokenIn, tokenOut, amountIn, amountOut,
   fee collected, route used, timestamp) for revenue tracking.

4. **Given** a user tries to swap with an input amount of zero or a token address that
   is not supported,
   **When** they request execution,
   **Then** the system rejects the request before a transaction is submitted, with a
   clear error message.

---

### User Story 3 — Compare All Available Routes (Priority: P2)

A power-user wants to see all available swap routes — not just the best one — so they
can make an informed decision (e.g., choosing a slightly worse price with lower gas,
or understanding why one DEX has better liquidity).

**Why this priority**: This adds transparency and trust, which increases conversion and
repeat usage. Not required for MVP launch but valuable soon after.

**Independent Test**: After requesting a quote, a user can expand a "Show all routes"
view and see each DEX's offered output amount, price impact, and gas estimate ranked
side by side.

**Acceptance Scenarios**:

1. **Given** a user requests a quote,
   **When** multiple DEX sources return results,
   **Then** the system presents all routes ranked by net output (after gas and fee),
   with the best route highlighted.

2. **Given** a user selects a non-best route manually,
   **When** they execute the swap,
   **Then** the system routes through the explicitly selected DEX, not the default best.

---

### Edge Cases

- What happens when a token pair has no liquidity on any supported DEX?
  → Return a clear "No route available" message; do not show a quote of zero.
- What happens when the user's wallet has insufficient balance for the input amount?
  → Surface a clear error before transaction submission; never let the on-chain tx fail for this reason.
- What happens when the Base RPC is unreachable?
  → Return a service-unavailable error with retry guidance; do not return stale or cached quotes silently.
- What happens with native ETH vs WETH differences?
  → The system handles ETH↔WETH wrapping transparently; the user always deals in ETH, not WETH.
- What happens when the protocol fee rate is updated?
  → In-flight quotes must reflect the fee rate at time of quote; execution uses the on-chain rate at
  time of execution. If these differ significantly, the user MUST be warned before confirming.

---

## Technical Architecture & Security Constraints

- Backend service MUST be implemented as a Node.js (v20+ LTS) + TypeScript service.
- The system operates on a non-custodial model for user funds during swaps.
- Private keys are generated per user and MUST be encrypted using AES-256-GCM before
  any database storage.
- Decryption keys MUST be managed strictly through environment variables.
- Private keys and decrypted key material MUST NOT be logged under any circumstance.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST accept a token pair (tokenIn address, tokenOut address) and a
  numeric input amount as the minimum required inputs for a quote.
- **FR-002**: Quote fetching MUST utilize off-chain DEX Aggregator API endpoints
  (e.g., 1inch API or 0x API) to source and compare routes on Base. The system MUST NOT
  query AMM smart contracts directly on-chain for quote discovery.
- **FR-003**: System MUST return the quote with the highest net output amount as the
  "best route" recommendation.
- **FR-004**: Every quote response MUST include: best output amount, price impact %,
  estimated gas cost (in USD equivalent), protocol fee amount, and source DEX name.
- **FR-005**: System MUST execute swaps atomically via a non-custodial smart contract
  on Base — no user funds may be held between transactions.
- **FR-006**: The smart contract MUST enforce a minimum output amount on every swap
  (on-chain slippage protection); if the actual output would be lower, the tx MUST revert.
- **FR-007**: Fee collection MUST be enforced by a custom, minimalist Solidity
  Proxy/Router contract. Execution MUST occur in a single atomic transaction where:
  (1) the user transaction is sent to the Proxy/Router, (2) the Proxy/Router executes
  the swap using external Aggregator API-provided calldata, (3) the Proxy/Router
  automatically deducts a 0.05% protocol fee from the OUTPUT token amount, (4) sends
  the fee to the treasury address, and (5) forwards the remaining 99.95% OUTPUT tokens
  to the user's wallet. No secondary fee transaction is permitted.
- **FR-008**: Every successful swap MUST emit an on-chain event containing: tokenIn,
  tokenOut, amountIn, amountOut, feeCollected, routeUsed, userAddress.
- **FR-009**: System MUST handle native ETH input transparently (wrap to WETH as needed
  for DEX routing without requiring user action).
- **FR-010**: System MUST display a high-price-impact warning when price impact exceeds 5%.

### Key Entities

- **SwapQuote**: tokenIn, tokenOut, amountIn, bestAmountOut, priceImpact, estimatedGasUSD,
  protocolFee, bestRoute, allRoutes[], quoteTimestamp, validForSeconds
- **SwapRoute**: sourceDex, path (intermediate tokens), amountOut, priceImpact, gasEstimate
- **SwapExecution**: txHash, tokenIn, tokenOut, amountIn, amountOut, feeCollected,
  routeUsed, userAddress, chainId, blockNumber, timestamp, status (success/reverted)

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users receive a complete best-price quote within **2 seconds** of submitting
  a valid token pair and amount (p95 under normal load).
- **SC-002**: The on-chain swap output is within **0.5% of the quoted amount** for any
  swap executed within 30 seconds of quote generation (excluding user-set slippage).
- **SC-003**: Protocol fee is correctly deducted and recorded in **100% of successful
  swap transactions** — verifiable by on-chain event log.
- **SC-004**: **95% or more** of submitted swap transactions complete successfully
  (not reverted due to system routing or contract errors — user-caused reverts excluded).
- **SC-005**: The system compares output from **at least 2 DEX sources** on every quote
  request — verifiable in the quote response payload.
- **SC-006**: Zero instances of user funds stuck in the aggregator contract — verifiable
  by on-chain balance monitoring.

---

## Assumptions

- Users have an EVM-compatible wallet (MetaMask, Coinbase Wallet, or WalletConnect-compatible)
  connected to the Base network.
- The default slippage tolerance is 0.5%; users may adjust it between 0.1% and 5%.
- Native ETH is treated as the user-facing asset; WETH wrapping/unwrapping is handled
  transparently by the system.
- v1 quote providers are off-chain DEX Aggregator APIs (1inch and/or 0x) for Base;
  the backend does not perform direct AMM smart-contract quote discovery on-chain.
- The protocol fee rate for v1 is fixed at **0.05%** of output amount, collected in the
  output token. The fee rate is stored on-chain and updatable only by the protocol owner
  (hardware wallet / multisig).
- Token allowlist: v1 supports a curated list of tokens (major stablecoins, ETH/WETH,
  WBTC, and top Base-native tokens). Arbitrary token pairs are out of scope for v1.
- Gas cost estimates are informational only and shown in USD; they do not affect
  routing decisions in v1.
- Multi-hop routes (e.g., ETH → USDC → DAI) are supported if a direct pair has
  insufficient liquidity, but limited to a maximum of 3 hops.
