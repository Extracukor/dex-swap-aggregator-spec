# Data Model: Frontend PoC Deployment

**Feature**: `003-frontend-poc-deployment`
**Date**: 2026-04-05

All types are TypeScript. All files live under `src/types/` or inline in their
respective service/hook file. No on-chain entities exist in this PoC.

---

## Core Types (`src/types/quote.ts`)

```typescript
/**
 * A single token in the curated allowlist.
 * Matches the shape planned for the full RoutingApiService integration (001).
 */
export interface TokenOption {
  address: string;    // checksummed EVM address (Base Sepolia)
  symbol: string;     // e.g. "ETH", "USDC"
  decimals: number;   // on-chain decimals (18 for ETH, 6 for USDC)
  logoURI?: string;   // optional CDN URL for token logo
}

/**
 * Input payload for MockQuoteService (and later, the real RoutingApiService).
 * Mirrors the GET /v1/quote query params defined in contracts/routing-api.md (001).
 */
export interface QuoteRequest {
  tokenIn: string;   // checksummed EVM address of input token
  tokenOut: string;  // checksummed EVM address of output token
  amountIn: string;  // human-readable amount as a string (e.g. "0.1" for 0.1 ETH)
}

/**
 * The quote result returned by MockQuoteService.
 * Shape is intentionally aligned with the planned real API response subset.
 * MUST contain exactly: { outputAmount, priceImpact, protocolFee }
 */
export interface QuoteResult {
  outputAmount: string;  // human-readable output amount (e.g. "180.123456")
  priceImpact: number;   // price impact as a decimal fraction (e.g. 0.042 = 4.2 bps)
  protocolFee: string;   // protocol fee in tokenOut units (e.g. "0.090062")
}

/**
 * The full async state managed by the useQuote hook.
 */
export interface QuoteState {
  data: QuoteResult | null;
  isLoading: boolean;
  error: string | null;
}
```

---

## Wallet Session State

Managed entirely by wagmi v2. No custom type is needed; use wagmi's exported types:

```typescript
// From wagmi — consumed in components via hooks
import { useAccount, useChainId, useSwitchChain } from 'wagmi';

// Relevant fields used in SwapWidget:
//   account.address: `0x${string}` | undefined
//   account.isConnected: boolean
//   chainId: number | undefined  (must equal 84532 for Base Sepolia)
```

---

## Curated Token Allowlist (`src/config/tokens.ts`)

```typescript
import { TokenOption } from '../types/quote';

/**
 * Hardcoded Base Sepolia testnet token allowlist for the PoC.
 * Addresses are Base Sepolia contract addresses (not mainnet).
 */
export const TOKEN_LIST: TokenOption[] = [
  {
    address: '0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE', // ETH sentinel
    symbol: 'ETH',
    decimals: 18,
  },
  {
    // USDC on Base Sepolia
    address: '0x036CbD53842c5426634e7929541eC2318f3dCF7e',
    symbol: 'USDC',
    decimals: 6,
  },
  {
    // WETH on Base Sepolia
    address: '0x4200000000000000000000000000000000000006',
    symbol: 'WETH',
    decimals: 18,
  },
];
```

---

## MockQuoteService Contract (`src/services/MockQuoteService.ts`)

```typescript
import type { QuoteRequest, QuoteResult } from '../types/quote';

/**
 * Simulates the Routing API (001) with a 1-second delay.
 * MUST return { outputAmount, priceImpact, protocolFee }.
 * Replace this import with RoutingApiService when 001 is ready.
 */
export async function getQuote(request: QuoteRequest): Promise<QuoteResult> {
  // Simulate network round-trip latency
  await new Promise<void>((resolve) => setTimeout(resolve, 1000));

  const amountInFloat = parseFloat(request.amountIn);

  if (isNaN(amountInFloat) || amountInFloat <= 0) {
    throw new Error('Invalid amountIn');
  }

  // Deterministic mock: treats all pairs as ETH→USDC equivalent at 1800 rate
  const grossOutput = amountInFloat * 1800;
  const fee = grossOutput * 0.0005; // 0.05% protocol fee

  return {
    outputAmount: grossOutput.toFixed(6),
    priceImpact: 0.042,              // 4.2 bps — realistic low-impact value
    protocolFee: fee.toFixed(6),
  };
}
```

---

## useQuote Hook Contract (`src/hooks/useQuote.ts`)

```typescript
/**
 * React hook signature — full implementation in Phase 2 tasks.
 */
export function useQuote(request: QuoteRequest | null): QuoteState;
```

**State transitions**:

```
null request         → { data: null, isLoading: false, error: null }
valid request        → { data: null, isLoading: true,  error: null }  (during delay)
service resolves     → { data: QuoteResult, isLoading: false, error: null }
service rejects      → { data: null, isLoading: false, error: "error message" }
request changes      → resets to loading state immediately
```

---

## Entity Relationships

```text
TokenOption ──(selected as)──► QuoteRequest.tokenIn
TokenOption ──(selected as)──► QuoteRequest.tokenOut
QuoteRequest ──(passed to)───► MockQuoteService.getQuote()
MockQuoteService ──(returns)─► QuoteResult
QuoteResult ──(held in)──────► QuoteState.data
QuoteState ──(rendered by)───► QuoteDisplay component
```

---

## Validation Rules

| Field | Rule |
|---|---|
| `QuoteRequest.tokenIn` | Must be a hex address from `TOKEN_LIST`; must differ from `tokenOut` |
| `QuoteRequest.tokenOut` | Must be a hex address from `TOKEN_LIST`; must differ from `tokenIn` |
| `QuoteRequest.amountIn` | Must parse to a positive finite float; empty string or `"0"` is invalid |
| `QuoteResult.priceImpact` | Decimal in range `[0, 1]`; display as `(priceImpact * 100).toFixed(2) + "%"` |
| `QuoteResult.protocolFee` | Non-negative string; display alongside `tokenOut.symbol` |
