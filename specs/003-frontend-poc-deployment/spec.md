# Feature Specification: Frontend PoC Deployment

**Feature Branch**: `003-frontend-poc-deployment`
**Created**: 2026-04-05
**Status**: Draft
**Input**: Prove the full frontend technology stack — React 19 + Vite + TypeScript +
wagmi v2 + viem v2 + RainbowKit + Tailwind CSS + shadcn/ui — end-to-end on Base Sepolia,
with a MockQuoteService replacing the real Routing API, deployed to Cloudflare Pages.

---

## Context & Purpose

Feature `001-core-swap-aggregation` (Routing Engine + Router contract) will not be
production-ready when the frontend team needs to start iterating on the UI/UX.
This PoC resolves the dependency by building the entire frontend stack against a
`MockQuoteService` that returns spec-compliant data with a simulated 1-second delay.

When `001` ships, the `MockQuoteService` is swapped for a real `RoutingApiService`
with zero changes to the component tree or wagmi config.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Connect Wallet and Receive Mock Quote (Priority: P1)

A developer or stakeholder opens the PoC app, connects a wallet on Base Sepolia via
RainbowKit, selects a token pair and amount, and within approximately 1 second sees
a mock quote displaying output amount, price impact percentage, and protocol fee.

**Why this priority**: This is the core proof that the full frontend stack is wired
correctly end-to-end — wallet → input → quote display. If this story works, the PoC
has achieved its purpose.

**Independent Test**: Open the deployed Cloudflare Pages URL, connect a wallet on Base
Sepolia, fill in the swap form, and observe a loading state followed by quote details.

**Acceptance Scenarios**:

1. **Given** the user opens the PoC app in a supported browser,
   **When** they click "Connect Wallet",
   **Then** RainbowKit's modal opens and the user can connect a wallet; wagmi v2 manages
   the connection state on Base Sepolia (chainId: `84532`).

2. **Given** the wallet is connected to Base Sepolia,
   **When** the user selects `tokenIn`, `tokenOut`, and enters a valid `amountIn`,
   **Then** a loading spinner appears for approximately 1 second (simulated API delay),
   followed by the quote result displaying:
   - `outputAmount` (formatted token amount string)
   - `priceImpact` (percentage, e.g. `"0.42%"`)
   - `protocolFee` (in tokenOut units, e.g. `"0.0021 USDC"`)

3. **Given** a quote is displayed,
   **When** the user changes `tokenIn`, `tokenOut`, or `amountIn`,
   **Then** the prior quote is cleared and the 1-second loading cycle restarts for
   the new inputs.

---

### User Story 2 — Input Guard and Network Switch Prompt (Priority: P1)

A user trying invalid inputs or connecting from the wrong network gets clear, immediate
UI feedback that prevents a broken experience.

**Why this priority**: Without guard rails, invalid usage generates confusing states.
These guards must be in place before any external testing or stakeholder demos.

**Independent Test**: Simulate edge cases — same token selected for both sides, zero
amount, wallet on Ethereum mainnet — and verify no quote request is triggered and
appropriate UI feedback is visible.

**Acceptance Scenarios**:

1. **Given** the user selects the same token for `tokenIn` and `tokenOut`,
   **When** both selectors have the same value,
   **Then** the "Get Quote" button is disabled and an inline message reads
   `"tokenIn and tokenOut must be different"`. No quote request is made.

2. **Given** the user enters `0` or an empty `amountIn`,
   **When** the form state reflects an invalid amount,
   **Then** the "Get Quote" button is disabled and an inline validation message reads
   `"Enter a valid amount"`.

3. **Given** the wallet is connected to a chain other than Base Sepolia (chainId ≠ 84532),
   **When** the swap form is rendered,
   **Then** a prominent banner prompts the user to switch to Base Sepolia; the quote
   form inputs are disabled until the network switch is confirmed via wagmi's
   `useSwitchChain` hook.

---

### User Story 3 — Cloudflare Pages Deployment (Priority: P1)

The PoC is deployed to Cloudflare Pages and accessible via a public HTTPS URL so
stakeholders can evaluate it without running a local dev server.

**Why this priority**: The Cloudflare Pages deployment pipeline is part of what this
PoC proves. Confirming that `vite build` + `wrangler pages deploy` (or git-connected
pipeline) works on the first pass means 005-frontend-dapp will deploy without friction.

**Independent Test**: Push to the `003-frontend-poc-deployment` branch and confirm a
Cloudflare Pages deployment URL is generated and serves the built app.

**Acceptance Scenarios**:

1. **Given** source code is pushed to the `003-frontend-poc-deployment` branch,
   **When** the Cloudflare Pages build pipeline runs `npm run build`,
   **Then** the `dist/` directory is generated and the Pages deployment succeeds
   with HTTP 200 at the published URL.

2. **Given** the Cloudflare Pages URL is opened in a browser,
   **When** the page loads,
   **Then** the React app renders without console errors, the RainbowKit "Connect
   Wallet" button is visible, and all Tailwind/shadcn/ui styles are applied correctly.

3. **Given** the deployed app,
   **When** US1 and US2 scenarios are performed end-to-end on the live deployment,
   **Then** all acceptance scenarios from US1 and US2 pass without a local dev server.

---

### Edge Cases

- What if `amountIn` is a non-numeric string? → Block at input level with HTML `type="number"` and TypeScript validation; no quote request fires.
- What if `MockQuoteService` throws an error? → `useQuote` hook catches the rejection and displays an error toast; the "Get Quote" button re-enables for retry.
- What if the wallet disconnects mid-flow? → wagmi's `useAccount` state transitions to `disconnected`; the swap form resets and prompts reconnection.
- What if the user switches to a non-Base-Sepolia network after a quote is displayed? → The displayed quote is invalidated and the network-switch banner reappears.
- What if the browser does not have a wallet extension installed? → RainbowKit's modal shows WalletConnect as a fallback option.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The project MUST be initialised with `npm create vite@latest . -- --template react-ts` (React 19 + TypeScript).
- **FR-002**: All source files MUST use `.ts` or `.tsx` extensions. No `.js` or `.jsx` files.
- **FR-003**: Wallet connectivity MUST use wagmi v2 and viem v2. RainbowKit MUST be the wallet modal. `ethers.js` and `web3modal` MUST NOT be installed.
- **FR-004**: Tailwind CSS MUST be installed and initialised (`tailwind.config.ts` + `postcss.config.js`) before shadcn/ui components are added.
- **FR-005**: UI components MUST use shadcn/ui primitives (Button, Select, Card, Skeleton, Sonner toast).
- **FR-006**: `MockQuoteService.ts` MUST:
  - Accept `{ tokenIn: string; tokenOut: string; amountIn: string }` as input.
  - Simulate a **1-second delay** using `await new Promise(resolve => setTimeout(resolve, 1000))`.
  - Return an object matching `{ outputAmount: string; priceImpact: number; protocolFee: string }`.
- **FR-007**: The wagmi config MUST define Base Sepolia (chainId `84532`) as the only supported chain using viem's `baseSepolia` chain definition.
- **FR-008**: The app MUST display the network-switch prompt when `useAccount().chainId !== 84532`.
- **FR-009**: The Cloudflare Pages deployment MUST be configured via a `wrangler.toml` at the project root specifying `pages_build_output_dir = "dist"`.
- **FR-010**: The PoC MUST NOT execute real on-chain transactions. It is a read/display PoC only.

### Key Entities

- **WalletSession**: Connected wallet state managed by wagmi — `{ address, chainId, isConnected }`.
- **TokenOption**: Curated testnet token entry used in selectors — `{ address: string; symbol: string; decimals: number; logoURI?: string }`.
- **QuoteRequest**: Input passed to `MockQuoteService` — `{ tokenIn: string; tokenOut: string; amountIn: string }`.
- **QuoteResult**: Return value from `MockQuoteService` — `{ outputAmount: string; priceImpact: number; protocolFee: string }`.
- **QuoteState**: React state managed by `useQuote` hook — `{ data: QuoteResult | null; isLoading: boolean; error: string | null }`.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: `MockQuoteService` responds with `{ outputAmount, priceImpact, protocolFee }` in ≥ 1000ms and ≤ 1200ms in 100% of invocations during local testing.
- **SC-002**: The Cloudflare Pages URL returns HTTP 200 and the React app renders without runtime errors in 100% of deployment smoke tests.
- **SC-003**: Zero `.js` or `.jsx` source files exist in the `src/` directory after project setup.
- **SC-004**: Zero `ethers`, `web3modal`, or wagmi v1 package references appear in `package.json`.
- **SC-005**: The network-switch prompt appears within 200ms of detecting a non-Base-Sepolia `chainId` on the connected wallet.
- **SC-006**: All three US1 and both US2 acceptance scenarios pass on the live Cloudflare Pages deployment.

---

## Assumptions

- The developer has a Cloudflare account and the Pages project is created (free tier).
- Base Sepolia testnet tokens for UI display are hardcoded in a curated allowlist; no on-chain token discovery is required for the PoC.
- The real Routing API (001) is not available during this PoC; `MockQuoteService` is the intentional and accepted temporary layer.
- `shadcn/ui` CLI (`npx shadcn@latest init`) is available and compatible with the Vite + React 19 + Tailwind setup.
- No CI/CD pipeline is required for the PoC; manual git-push to Cloudflare Pages is acceptable.
