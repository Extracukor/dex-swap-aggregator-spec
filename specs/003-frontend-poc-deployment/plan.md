# Implementation Plan: 003 Frontend PoC Deployment

**Branch**: `003-frontend-poc-deployment` | **Date**: 2026-04-05 | **Spec**: `specs/003-frontend-poc-deployment/spec.md`

---

## Summary

Build and deploy a frontend-only PoC that proves the full required stack end-to-end on Base Sepolia:
React 19 + Vite + TypeScript + wagmi v2 + viem v2 + RainbowKit v2 + Tailwind CSS + shadcn/ui,
using a strict mock quoting layer (`MockQuoteService`) instead of the real routing backend.

The PoC is intentionally read/display-only (no real swap execution) and is deployed to Cloudflare Pages.

---

## Technical Context

**Language/Version**: TypeScript 5.x (application and config source in `.ts` / `.tsx`)
**Primary Dependencies**:
- `react@19`, `react-dom@19`, `vite`, `@vitejs/plugin-react`
- `wagmi@^2`, `viem@^2`, `@rainbow-me/rainbowkit@^2`, `@tanstack/react-query@^5`
- `tailwindcss`, `postcss`, `autoprefixer`, `class-variance-authority`, `clsx`, `tailwind-merge`
- shadcn/ui components via CLI-generated local components
- `sonner` for toast notifications
**Storage**: N/A (no backend persistence in this PoC)
**Testing**: Vitest + React Testing Library
**Target Platform**: Web browser (Cloudflare Pages)
**Project Type**: Frontend web app (PoC)
**Performance Goals**:
- Quote loading state starts immediately and resolves in 1000-1200ms
- First render under typical Vite SPA baseline for testnet demo usage
**Constraints**:
- TypeScript-only source files (`.ts` / `.tsx`)
- No `ethers`, no `web3modal`, no wagmi v1 APIs
- Base Sepolia (`chainId = 84532`) as only supported chain
- No real on-chain swap transactions
**Scale/Scope**: Stakeholder/demo PoC (single-page dApp experience)

---

## FR-001 to FR-010 Compliance Matrix

| FR | Requirement | Plan Decision | Verification |
|---|---|---|---|
| FR-001 | Initialize with Vite React TypeScript template | Run `npm create vite@latest . -- --template react-ts` as first step | `package.json`, `tsconfig.json`, Vite scaffold present |
| FR-002 | `.ts`/`.tsx` source only, no `.js`/`.jsx` source | Use TypeScript for all app/hook/service/config source files | `find src -type f \( -name "*.js" -o -name "*.jsx" \)` returns empty |
| FR-003 | Wallet connectivity via wagmi v2 + viem v2 + RainbowKit; no ethers/web3modal | Install only wagmi/viem/RainbowKit stack and configure wagmi provider | `npm ls ethers web3modal` fails/not found; wallet connects via RainbowKit |
| FR-004 | Tailwind initialized before shadcn/ui | Install and initialize Tailwind first, then run shadcn init/add commands | Tailwind styles active and shadcn components render correctly |
| FR-005 | UI built from shadcn/ui primitives | Use shadcn Button, Select, Card, Skeleton and Sonner toast | UI review + component imports from `src/components/ui/*` |
| FR-006 | `MockQuoteService` input/delay/output contract | Implement typed service with exact 1s delay and exact output object keys | Unit test for timing and output shape |
| FR-007 | Base Sepolia only in wagmi config | Configure `chains: [baseSepolia]` only | Runtime chain list contains only 84532 |
| FR-008 | Network switch prompt for wrong chain | Render banner + disable form + use `useSwitchChain` action | Manual scenario test with non-84532 chain |
| FR-009 | Cloudflare Pages setup via `wrangler.toml` with `dist` output | Add root `wrangler.toml` with `pages_build_output_dir = "dist"` | Pages build/deploy succeeds using Vite output |
| FR-010 | No real on-chain transaction execution | No swap/sign/send call paths implemented; quote display only | Code review confirms no `writeContract`/`sendTransaction` path |

---

## Project Structure

### Documentation (this feature)

```text
specs/003-frontend-poc-deployment/
├── plan.md              ← this file
├── research.md
├── data-model.md
├── quickstart.md        ← to be added
├── contracts/
│   └── mock-service-contract.md
└── tasks.md             ← generated in Phase 2
```

### Source Code (new frontend app; TypeScript only)

```text
apps/frontend-poc/
├── package.json
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── vite.config.ts
├── tailwind.config.ts
├── postcss.config.ts
├── components.json
├── wrangler.toml
├── index.html
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── index.css
    ├── lib/
    │   ├── cn.ts
    │   └── utils.ts
    ├── config/
    │   ├── chains.ts
    │   ├── wagmi.ts
    │   └── tokens.ts
    ├── types/
    │   └── quote.ts
    ├── services/
    │   └── MockQuoteService.ts
    ├── hooks/
    │   ├── useWalletNetwork.ts
    │   └── useQuote.ts
    ├── components/
    │   ├── layout/
    │   │   └── AppShell.tsx
    │   ├── wallet/
    │   │   ├── WalletConnectButton.tsx
    │   │   └── NetworkGuardBanner.tsx
    │   ├── swap/
    │   │   ├── SwapWidget.tsx
    │   │   ├── TokenSelect.tsx
    │   │   ├── AmountInput.tsx
    │   │   ├── QuoteButton.tsx
    │   │   └── QuoteResultCard.tsx
    │   └── ui/
    │       ├── button.tsx
    │       ├── card.tsx
    │       ├── select.tsx
    │       └── skeleton.tsx
    └── test/
        ├── MockQuoteService.test.ts
        └── useQuote.test.tsx
```

---

## Exact Component Tree

```text
Root
└── Providers
    ├── QueryClientProvider
    ├── WagmiProvider
    └── RainbowKitProvider
        └── AppShell
            ├── Header
            │   ├── Brand
            │   └── WalletConnectButton
            └── Main
                ├── NetworkGuardBanner
                └── SwapWidget
                    ├── TokenSelect (tokenIn)
                    ├── TokenSelect (tokenOut)
                    ├── AmountInput
                    ├── ValidationMessage
                    ├── QuoteButton
                    └── QuoteResultCard
                        ├── Skeleton (loading)
                        ├── OutputAmountRow
                        ├── PriceImpactRow
                        └── ProtocolFeeRow
            └── Toaster (Sonner)
```

---

## Setup & Initialization Steps

## 1) Bootstrap (FR-001, FR-002)

```bash
mkdir -p apps/frontend-poc && cd apps/frontend-poc
npm create vite@latest . -- --template react-ts
npm install
```

## 2) Wallet stack (FR-003, FR-007)

```bash
npm install wagmi viem @rainbow-me/rainbowkit @tanstack/react-query
```

Explicitly forbidden packages:
- `ethers`
- `web3modal`

## 3) Tailwind first, then shadcn/ui (FR-004, FR-005)

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Immediately convert generated config files to TypeScript naming/format used by this plan:
- `tailwind.config.js` → `tailwind.config.ts`
- `postcss.config.js` → `postcss.config.ts`

Add Tailwind directives to `src/index.css`:
- `@tailwind base;`
- `@tailwind components;`
- `@tailwind utilities;`

Initialize shadcn/ui after Tailwind is active:

```bash
npx shadcn@latest init
npx shadcn@latest add button select card skeleton
npm install sonner
```

## 4) Cloudflare Pages config (FR-009)

Create `wrangler.toml`:

```toml
name = "dex-swap-aggregator-frontend-poc"
compatibility_date = "2026-04-05"
pages_build_output_dir = "dist"
```

Build/deploy commands:

```bash
npm run build
npx wrangler pages deploy dist
```

---

## Implementation Design

## Wallet and Network Guard

- `config/wagmi.ts` sets only `baseSepolia` in chain list.
- `NetworkGuardBanner.tsx` checks connected `chainId`.
- If `chainId !== 84532`, form controls are disabled and a switch-chain CTA is shown.
- `useSwitchChain` triggers Base Sepolia switch flow.

## Quote Flow

- `SwapWidget.tsx` captures `tokenIn`, `tokenOut`, `amountIn`.
- Client validation blocks invalid requests:
  - Same token in/out
  - Missing or non-positive amount
- `useQuote` invokes `MockQuoteService.getQuote` only when request is valid and wallet/network state permits.
- During call, `QuoteResultCard` shows Skeleton for 1 second simulation window.

---

## MockQuoteService Implementation (FR-006)

File: `src/services/MockQuoteService.ts`

```ts
import type { QuoteRequest, QuoteResult } from '../types/quote';

export async function getQuote(request: QuoteRequest): Promise<QuoteResult> {
  const { tokenIn, tokenOut, amountIn } = request;

  if (!tokenIn || !tokenOut || tokenIn === tokenOut) {
    throw new Error('Invalid token pair');
  }

  const amount = Number(amountIn);
  if (!Number.isFinite(amount) || amount <= 0) {
    throw new Error('Invalid amountIn');
  }

  await new Promise<void>((resolve) => setTimeout(resolve, 1000));

  const grossOutput = amount * 1800;
  const protocolFeeValue = grossOutput * 0.0005;

  return {
    outputAmount: grossOutput.toFixed(6),
    priceImpact: 0.042,
    protocolFee: protocolFeeValue.toFixed(6),
  };
}
```

Return object is exactly:
- `outputAmount`
- `priceImpact`
- `protocolFee`

No additional keys.

---

## Testing Strategy

- Unit test `MockQuoteService`:
  - Asserts duration is >= 1000ms and <= 1200ms
  - Asserts exact key shape and value types
  - Asserts invalid amount/token scenarios throw expected errors
- Hook/component tests:
  - Loading Skeleton appears during delay
  - Network guard appears when chain is not Base Sepolia
  - Get Quote button disabled on invalid input
- Deployment smoke test:
  - Cloudflare Pages URL loads with 200
  - RainbowKit connect button visible
  - Quote flow works end-to-end with mock response

---

## Phase Plan

## Phase 0: Research

Completed in `research.md`.

## Phase 1: Design & Contracts

Completed artifacts:
- `data-model.md`
- `contracts/mock-service-contract.md`

Remaining artifact:
- `quickstart.md`

## Phase 2: Implementation Plan

Generate `tasks.md` with atomic tasks mapped to FR-001..FR-010.

---

## Risk Register

- Risk: Wallet not on Base Sepolia during demos.
  - Mitigation: Persistent network guard banner + one-click switch action.
- Risk: Tailwind/shadcn mismatch during setup.
  - Mitigation: Enforce Tailwind initialization before shadcn CLI.
- Risk: Type drift between mock and future API.
  - Mitigation: Keep `QuoteRequest`/`QuoteResult` contract centralized in `types/quote.ts`.

---

## Constitution Check (Rigorous)

This gate is evaluated against `.specify/memory/constitution.md` and FR-001..FR-010.

### Principle I — On-Chain Safety First

Status: PASS

- PoC does not execute on-chain swaps (FR-010).
- No contract deployment/change in this feature.
- No user funds are handled or moved.

### Principle II — Revenue-Driven Simplicity (YAGNI)

Status: PASS

- Scope is intentionally minimal: wallet connect + quote display + deployment validation.
- No speculative advanced routing, execution, or analytics features included.

### Principle III — L2-First, EVM-Compatible Expansion

Status: PASS

- Only Base Sepolia is enabled in wagmi config (FR-007).
- Network guard enforces expected chain (FR-008).

### Principle IV — API-First Routing Engine

Status: PASS

- Mock service follows a strict typed contract aligned with routing interface.
- Future migration path is import-level replacement, preserving API contract discipline.

### Principle V — Observability & Revenue Monitoring

Status: PASS (PoC-scope)

- Feature is frontend-only and non-revenue-generating.
- No contradiction with required health/revenue obligations for backend/mainnet features.

### Principle VI — Swap Execution Boundaries & Fee Integrity

Status: PASS

- Frontend does not trigger real swap execution in this PoC (FR-010).
- No backend custody or key handling introduced.
- Fee shown is mock-display only, not executed.

### Constraints & Standards Check

Status: PASS

- Frontend stack remains React + Vite + TypeScript.
- Wallet stack is wagmi v2 + viem v2 + RainbowKit v2 only.
- No forbidden packages (`ethers`, `web3modal`) in plan dependencies.
- Cloudflare Pages deployment configured via `wrangler.toml` (FR-009).

### Final Gate Decision

PASS — Plan is constitution-compliant and satisfies FR-001 through FR-010 without justified violations.

## Complexity Tracking

No constitution violations; table intentionally empty.
