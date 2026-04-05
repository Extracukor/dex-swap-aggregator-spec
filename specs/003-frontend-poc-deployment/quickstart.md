# Quickstart Validation: Frontend PoC Deployment

**Feature**: `003-frontend-poc-deployment`
**Date**: 2026-04-05

This quickstart validates the full frontend PoC flow end-to-end with the required stack:
React 19 + Vite + TypeScript + wagmi v2 + viem v2 + RainbowKit + Tailwind + shadcn/ui,
using `MockQuoteService` on Base Sepolia.

---

## Prerequisites

- [ ] Node.js 20+ and npm installed
- [ ] Browser wallet installed (MetaMask or equivalent)
- [ ] Wallet can connect to Base Sepolia (`chainId: 84532`)
- [ ] Cloudflare account (for Pages deployment validation)

---

## Setup (FR-001, FR-002, FR-003, FR-004, FR-005, FR-009)

From repository root:

```bash
mkdir -p apps/frontend-poc
cd apps/frontend-poc
npm create vite@latest . -- --template react-ts
npm install

npm install wagmi viem @rainbow-me/rainbowkit @tanstack/react-query
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

npx shadcn@latest init
npx shadcn@latest add button select card skeleton
npm install sonner
```

Create `wrangler.toml`:

```toml
name = "dex-swap-aggregator-frontend-poc"
compatibility_date = "2026-04-05"
pages_build_output_dir = "dist"
```

Forbidden packages check:

```bash
npm ls ethers web3modal
```

Expected:
- command indicates packages are not present

TypeScript-only source check:

```bash
find src -type f \( -name "*.js" -o -name "*.jsx" \)
```

Expected:
- no results

---

## Scenario 1 — Wallet Connect on Base Sepolia (US1, FR-003, FR-007)

**Goal**: Confirm RainbowKit + wagmi v2 + viem v2 integration and chain support.

1. Start dev server:

```bash
npm run dev
```

2. Open shown local URL.
3. Click `Connect Wallet`.
4. Connect wallet via RainbowKit.

Expected:
- [ ] RainbowKit modal opens
- [ ] Wallet connects successfully
- [ ] Connected chain is Base Sepolia (`84532`)
- [ ] No `ethers` or `web3modal` usage is required anywhere

---

## Scenario 2 — Mock Quote Success Path (US1, FR-006, SC-001)

**Goal**: Confirm 1-second simulated quote with exact response shape fields.

1. With wallet connected on Base Sepolia, select `tokenIn`, `tokenOut`, and a valid `amountIn`.
2. Trigger `Get Quote`.
3. Observe loading skeleton/spinner.
4. Wait for quote to appear.

Expected:
- [ ] Loading state appears immediately
- [ ] Quote resolves in >= 1000ms and <= 1200ms
- [ ] Displayed quote includes:
  - [ ] `outputAmount`
  - [ ] `priceImpact`
  - [ ] `protocolFee`
- [ ] No additional mock fields are required by UI contract

---

## Scenario 3 — Input Validation Guards (US2, FR-008)

**Goal**: Invalid inputs must block quote calls and guide the user.

### 3A: Same token pair

1. Select identical token for `tokenIn` and `tokenOut`.

Expected:
- [ ] Get Quote button disabled
- [ ] Inline message: `tokenIn and tokenOut must be different`
- [ ] No quote request fired

### 3B: Invalid amount

1. Enter empty amount or `0`.

Expected:
- [ ] Get Quote button disabled
- [ ] Inline message: `Enter a valid amount`
- [ ] No quote request fired

---

## Scenario 4 — Network Switch Prompt (US2, FR-008, SC-005)

**Goal**: App must force Base Sepolia context for quote interactions.

1. Connect wallet on a non-Base-Sepolia chain (for example Ethereum mainnet).
2. Open/return to swap form.

Expected:
- [ ] Network guard banner appears in <= 200ms
- [ ] Prompt indicates switching to Base Sepolia is required
- [ ] Swap inputs and quote action are disabled until switched
- [ ] Switch action uses wagmi `useSwitchChain`

---

## Scenario 5 — Cloudflare Pages Deploy (US3, FR-009, SC-002)

**Goal**: Validate PoC is publicly reachable on Cloudflare Pages.

Build locally:

```bash
npm run build
```

Expected:
- [ ] `dist/` is generated

Deploy:

```bash
npx wrangler pages deploy dist
```

Expected:
- [ ] Deployment succeeds
- [ ] Public HTTPS URL is returned
- [ ] URL opens with HTTP 200
- [ ] RainbowKit connect button visible
- [ ] Tailwind/shadcn styling is applied

---

## Scenario 6 — Read/Display-Only Safety Check (FR-010)

**Goal**: Confirm PoC does not execute real on-chain swaps.

Verify code paths and runtime behavior:
- [ ] No `writeContract` usage in quote flow
- [ ] No `sendTransaction` call in quote flow
- [ ] UI exposes quote/display interaction only

Optional code scan:

```bash
rg "writeContract|sendTransaction|walletClient\.send" src
```

Expected:
- no swap execution path tied to this PoC flow

---

## Pass Criteria

All scenarios must pass:

- Scenario 1 (wallet connection)
- Scenario 2 (mock quote with 1s delay and exact output fields)
- Scenario 3 (input validation guards)
- Scenario 4 (network switch guard)
- Scenario 5 (Cloudflare Pages deployment)
- Scenario 6 (no real transaction execution)

When all pass, the 003 frontend PoC is validated against FR-001..FR-010 and SC-001..SC-006.
