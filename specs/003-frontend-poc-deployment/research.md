# Research: Frontend PoC Deployment

**Feature**: `003-frontend-poc-deployment`
**Date**: 2026-04-05

All technology decisions resolved below. No `NEEDS CLARIFICATION` items remain.

---

## Decision 1: Project Bootstrap

**Decision**: `npm create vite@latest . -- --template react-ts`

**Rationale**: The constitution explicitly mandates React 19 + Vite and TypeScript for
the frontend. The `react-ts` Vite template emits a fully-typed project with `.tsx`
entry points, `tsconfig.json`, and Vite 5 out of the box. This is the fastest compliant
starting point.

**Alternatives Considered**:
- Next.js: Server-side rendering adds unnecessary complexity for a client-only wallet
  dApp; SSR does not compose cleanly with wagmi hooks that require browser globals.
- Create React App: Deprecated; lacks Vite's fast HMR and native ESM build chain.
- Plain `vite` + manual React setup: More steps, same outcome; the template is idiomatic.

---

## Decision 2: Wallet Library

**Decision**: wagmi v2 + viem v2 + RainbowKit v2 + `@tanstack/react-query` v5

**Rationale**: The constitution (Constraints & Standards) mandates wagmi v2 and viem v2.
RainbowKit v2 is the spec-required wallet modal (FR-004 in spec `005-frontend-dapp` and
FR-003 here). `@tanstack/react-query` v5 is wagmi v2's required peer for async state
management. `ethers.js` and `web3modal` are explicitly prohibited (FR-003, SC-004).

**Alternatives Considered**:
- `ethers.js` v6: Ruled out — constitution mandates viem; ethers.js is a prohibited install.
- Web3Modal: Ruled out — explicitly forbidden by FR-003.
- ConnectKit: Valid RainbowKit alternative, but RainbowKit is the spec-mandated choice.
- wagmi v1: Ruled out by constitution Constraint ("React 19 + wagmi v2 + viem v2")
  and spec SC-005 ("Zero legacy wagmi v1 hook usage").

---

## Decision 3: UI Framework

**Decision**: Tailwind CSS v3 + shadcn/ui

**Rationale**: The constitution mandates Tailwind CSS + shadcn/ui (Constraints &
Standards, `005-frontend-dapp` FR-002). Tailwind must be initialised before shadcn/ui
because the shadcn CLI expects `tailwind.config.ts` and a CSS file with Tailwind
directives. Tailwind v3 (not v4 alpha) is chosen for stability and full shadcn/ui
compatibility.

**Tailwind init steps** (order-sensitive):
1. `npm install -D tailwindcss postcss autoprefixer`
2. `npx tailwindcss init -p` → generates `tailwind.config.js` (rename to `.ts`)
3. Add `@tailwind base/components/utilities` directives to `src/index.css`
4. `npx shadcn@latest init` → configures `components.json`, updates `tailwind.config.ts`

**Alternatives Considered**:
- Tailwind v4: Still alpha/beta as of 2026-04-05; shadcn/ui CLI compatibility is not
  guaranteed. Rejected in favour of stability.
- CSS Modules: No component-level design system; more boilerplate per component.
- MUI / Chakra: Non-shadcn component libraries; conflict with constitution's shadcn mandate.

---

## Decision 4: MockQuoteService Contract

**Decision**: `MockQuoteService.ts` returns `{ outputAmount: string; priceImpact: number; protocolFee: string }` after a 1-second `setTimeout` delay.

**Rationale**: The PoC must validate the component tree without the real Routing API
(001). The mock return shape is deliberately aligned with the planned `QuoteResult`
interface from `contracts/routing-api.md` in feature 001 so the swap from mock to real
service requires only a one-line import change. The 1-second delay simulates realistic
API round-trip latency and ensures loading states are exercised in the PoC.

**Mock output values**: Deterministic based on input to allow snapshot testing:
- `outputAmount`: `(parseFloat(amountIn) * 1800).toFixed(6)` (ETH→USDC proxy)
- `priceImpact`: `0.042` (4.2 bps — within normal range)
- `protocolFee`: `(parseFloat(amountIn) * 1800 * 0.0005).toFixed(6)` (0.05% fee)

**Alternatives Considered**:
- MSW (Mock Service Worker): Powerful but adds complexity; overkill for a single-function
  mock that is explicitly temporary.
- hardcoded constants: Non-deterministic relative to input — hides integration bugs early.
- Random values: Make snapshot tests non-reproducible; rejected.

---

## Decision 5: Cloudflare Pages Configuration

**Decision**: `wrangler.toml` at project root with `pages_build_output_dir = "dist"`.

**Rationale**: The constitution mandates Cloudflare Pages for frontend hosting. The
`wrangler.toml` approach is declarative (infrastructure-as-code), version-controlled,
and avoids manual dashboard configuration. Vite's default build output is `dist/`,
which maps directly to Cloudflare Pages' expected directory.

**wrangler.toml minimum viable configuration**:

```toml
name = "dex-swap-aggregator-poc"
compatibility_date = "2026-04-05"
pages_build_output_dir = "dist"
```

**Cloudflare Pages build settings** (via dashboard or wrangler):
- Build command: `npm run build`
- Output directory: `dist`
- Node.js version: `20`

**Alternatives Considered**:
- Manual dashboard config only: Not version-controlled; breaks the "git push deploys
  automatically" requirement from the constitution.
- `_redirects` file for SPA routing: Needed alongside `wrangler.toml` if client-side
  routing beyond single-page is used; noted but not required for this PoC.

---

## Decision 6: Testing

**Decision**: Vitest + React Testing Library (smoke tests only for PoC)

**Rationale**: Vitest is the idiomatic Vite-native test runner. React Testing Library
provides component-level unit tests without full browser launch. For the PoC scope,
only smoke tests for `MockQuoteService` and `useQuote` hook are required — no e2e
Playwright/Cypress setup.

**Alternatives Considered**:
- Jest: Requires babel transform config; Vitest is a drop-in with native Vite integration.
- Playwright: E2e tests are out of scope for a PoC; the quickstart covers manual
  validation on the deployed URL.
