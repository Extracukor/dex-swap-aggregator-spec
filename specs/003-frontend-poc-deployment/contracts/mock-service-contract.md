# Mock Service Contract

**Feature**: `003-frontend-poc-deployment`
**Date**: 2026-04-05

This document defines the interface contract for `MockQuoteService`, the temporary
stand-in for the real Routing API defined in `specs/001-core-swap-aggregation/contracts/routing-api.md`.

The mock contract is intentionally shape-compatible with the real API so that
swapping implementations requires only a one-line import change.

---

## Function Signature

```typescript
// src/services/MockQuoteService.ts
export async function getQuote(request: QuoteRequest): Promise<QuoteResult>
```

---

## Input: `QuoteRequest`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `tokenIn` | `string` | Non-empty hex address from `TOKEN_LIST` | Input token EVM address |
| `tokenOut` | `string` | Non-empty hex address from `TOKEN_LIST`; must differ from `tokenIn` | Output token EVM address |
| `amountIn` | `string` | Parses to a positive finite float | Input amount in human-readable units (e.g. `"0.1"` for 0.1 ETH) |

---

## Output: `QuoteResult`

| Field | Type | Description |
|---|---|---|
| `outputAmount` | `string` | Gross output in human-readable tokenOut units (before fee deduction) |
| `priceImpact` | `number` | Estimated price impact as a decimal fraction (e.g. `0.042` = 4.2 bps). Display as `(value * 100).toFixed(2) + "%"` |
| `protocolFee` | `string` | Protocol fee amount in tokenOut human-readable units (0.05% of `outputAmount`) |

---

## Behaviour Contract

| Behaviour | Specification |
|---|---|
| **Delay** | MUST await exactly 1000ms via `setTimeout` before returning or throwing |
| **Determinism** | Same `QuoteRequest` inputs MUST always produce the same `QuoteResult` |
| **Error path** | MUST throw `new Error('Invalid amountIn')` if `amountIn` is `NaN`, `0`, or negative |
| **No side effects** | MUST NOT make network calls, write to storage, or mutate external state |
| **Pure output** | MUST NOT include fields beyond `{ outputAmount, priceImpact, protocolFee }` in the returned object |

---

## Mock Calculation Logic

```text
grossOutput   = parseFloat(amountIn) × 1800       // proxy rate for any pair
protocolFee   = grossOutput × 0.0005              // 0.05% = 5 bps
outputAmount  = grossOutput.toFixed(6)
priceImpact   = 0.042                             // fixed: 4.2 bps
protocolFee   = (grossOutput × 0.0005).toFixed(6)
```

The fixed rate of 1800 is a proxy for ETH→USDC equivalence and is used consistently
across all token pairs in the PoC to keep snapshot tests reproducible.

---

## Migration Path to Real API

When `001-core-swap-aggregation` Routing Engine is ready, replace the mock with:

```typescript
// Before (PoC):
import { getQuote } from './MockQuoteService';

// After (production):
import { getQuote } from './RoutingApiService';
```

`RoutingApiService.getQuote()` MUST implement the same `(QuoteRequest) => Promise<QuoteResult>`
signature. The `useQuote` hook, `SwapWidget`, and `QuoteDisplay` components require
**zero changes**.

---

## Error Codes

| Code | Trigger | Message |
|---|---|---|
| `INVALID_AMOUNT` | `amountIn` is `NaN`, empty, `"0"`, or negative | `"Invalid amountIn"` |

The `useQuote` hook catches all thrown errors and surfaces them as `QuoteState.error: string`.
No other error codes are defined for the PoC.
