# Feature Specification: Frontend dApp

**Feature Branch**: `005-frontend-dapp`
**Created**: 2026-04-05
**Status**: Draft
**Input**: User description: "Primary web interface (dApp) for the DEX Swap Aggregator, strictly following the Constitution (React 19, Vite)."

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Connect Wallet and Get Quote (Priority: P1)

A user wants to connect a wallet on Base, choose tokenIn/tokenOut from a curated list,
enter an amount, and receive a reliable quote from the backend before deciding to swap.

**Why this priority**: Quote visibility is the main decision point. Without it, the dApp
cannot convert visitors into swap executions.

**Independent Test**: A user connects wallet, selects token pair and amount, and sees
quote details sourced from the backend API without performing any on-client routing math.

**Acceptance Scenarios**:

1. **Given** the user opens the dApp on a supported browser,
   **When** they click connect wallet,
   **Then** RainbowKit opens and the wallet connects through wagmi v2 on Base network.

2. **Given** wallet is connected,
   **When** the user selects tokenIn, tokenOut, and enters amount,
   **Then** the frontend requests a quote from the custom Node.js Routing API and displays
   output amount, route summary, protocol fee, price impact, and estimated gas.

3. **Given** a quote response is received,
   **When** the UI renders quote details,
   **Then** all values are shown as backend-provided data and no client-side heavy routing
   calculations are performed.

---

### User Story 2 - Review Risk and Execute Swap (Priority: P1)

A user wants to review the quote quality signals (price impact and gas estimate), then
execute the swap with wallet signature and broadcast from the dApp flow.

**Why this priority**: Swap execution is the core value event and directly tied to protocol revenue.

**Independent Test**: Starting from a valid quote, a user can click Swap, approve signature,
and submit the transaction through wallet flow.

**Acceptance Scenarios**:

1. **Given** a valid quote is displayed,
   **When** the user reviews price impact and gas estimate,
   **Then** the UI clearly highlights risk indicators before execution.

2. **Given** the user confirms swap,
   **When** they click Swap,
   **Then** wagmi v2 initiates wallet signature flow and the transaction is broadcasted.

3. **Given** the transaction hash is returned,
   **When** the broadcast succeeds,
   **Then** the dApp displays submission success feedback and a block explorer link.

---

### User Story 3 - Clear Error Feedback (Priority: P2)

A user needs understandable feedback when something goes wrong so they can recover
without confusion or support tickets.

**Why this priority**: Clear errors reduce drop-off and support burden.

**Independent Test**: Simulate rejection, insufficient balance, and backend API failure,
and verify clear toast notifications and non-blocking recovery UX.

**Acceptance Scenarios**:

1. **Given** wallet signature is requested,
   **When** the user rejects the signature,
   **Then** the dApp shows a clear toast explaining that the transaction was canceled by user.

2. **Given** the user has insufficient token or gas balance,
   **When** they attempt to execute swap,
   **Then** the dApp shows an insufficient balance toast with actionable guidance.

3. **Given** the quote API returns an error or timeout,
   **When** quote retrieval fails,
   **Then** the dApp shows an API error toast and allows retry without page refresh.

---

### Edge Cases

- What happens if the wallet is connected to a non-Base network? -> Prompt network switch to Base before allowing quote or swap.
- What happens if tokenIn and tokenOut are the same? -> Disable quote request and show a validation message.
- What happens if amount is zero, negative, or malformed? -> Block submission and show inline input validation.
- What happens if the quote expires before user clicks Swap? -> Invalidate the quote and request a fresh quote.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The frontend tech stack MUST be React 19 + Vite.
- **FR-002**: The frontend UI layer MUST use Tailwind CSS and shadcn/ui for minimalist, accessible components.
- **FR-003**: Web3 integration MUST strictly use wagmi v2 and viem v2 for wallet connectivity and contract interactions.
- **FR-004**: RainbowKit MUST be used as the wallet connection modal.
- **FR-005**: The frontend MUST NOT use legacy wagmi v1 hooks.
- **FR-006**: The frontend MUST act as a thin client and fetch swap quotes, routes, gas, and fee breakdown strictly from the custom Node.js Routing API.
- **FR-007**: The frontend MUST NOT perform heavy routing math or route optimization logic client-side.
- **FR-008**: The primary user flow MUST support: wallet connect on Base, curated token selection, amount input, backend quote fetch, quote review, swap click, wallet signature, and transaction broadcast.
- **FR-009**: The frontend MUST present clear toast notifications for user signature rejection, insufficient balance, and backend API errors.
- **FR-010**: The frontend MUST enforce Base network requirement before allowing swap execution.

### Key Entities

- **WalletSession**: Connected wallet state including address, chainId, and connection status.
- **TokenOption**: Curated token metadata used in selectors (address, symbol, decimals, logo).
- **QuoteRequest**: Input payload sent to Routing API (tokenIn, tokenOut, amountIn, userAddress).
- **QuoteResponseViewModel**: Backend-provided quote data rendered by UI (amountOut, route, protocolFee, gasEstimate, priceImpact, expiry).
- **SwapSubmission**: Execution state for signing and broadcasting (status, txHash, errorCode, errorMessage).
- **ToastEvent**: User-facing notification model (type, title, message, action).

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 95% or more of users can complete connect->quote flow in under 30 seconds on standard desktop connection.
- **SC-002**: 99% of displayed quote values are traceable to the latest successful Routing API response with no client-side recomputation drift.
- **SC-003**: 95% or more of eligible users can complete quote->signature->broadcast flow on first attempt.
- **SC-004**: 100% of simulated user rejection, insufficient balance, and API error cases produce explicit toast notifications.
- **SC-005**: Zero legacy wagmi v1 hook usage is detected in the frontend codebase.

---

## Assumptions

- Routing API endpoints are available and versioned before frontend integration begins.
- Curated token list is maintained by backend or shared config and available to frontend at runtime.
- Wallet providers used by target users support Base network switching through RainbowKit/wagmi.
- The frontend does not custody private keys and relies on user wallet signing.
