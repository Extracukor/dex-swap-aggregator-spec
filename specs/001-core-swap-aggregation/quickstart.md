# Quickstart Validation: Core Swap Aggregation

**Feature**: `001-core-swap-aggregation`
**Date**: 2026-04-02

These are the minimum steps to verify the feature works end-to-end on Base Sepolia
testnet before mainnet deployment.

---

## Prerequisites

- [ ] Routing engine running locally or on staging: `http://localhost:3000`
- [ ] `AggregatorRouter` deployed to Base Sepolia, address recorded in `.env`
- [ ] Test wallet funded with Base Sepolia ETH and test USDC
- [ ] Browser with MetaMask connected to Base Sepolia (chainId: `84532`)

---

## Scenario 1 — ETH → USDC Quote (US1, SC-001, SC-005)

**Goal**: Verify quote returns in <2s with ≥2 DEX sources compared.

```bash
curl "http://localhost:3000/v1/quote?tokenIn=0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE&tokenOut=0x036CbD53842c5426634e7929541eC2318f3dCF7e&amountIn=100000000000000000&chainId=84532"
```

Expected response:

- `allRoutes` array has length ≥ 2 ✓
- `bestRoute.dexId` is `"uniswap_v3"` or `"aerodrome"` ✓
- `protocolFeeBps` equals `5` ✓
- `highPriceImpact` is `false` for 0.1 ETH ✓
- `validForSeconds` equals `30` ✓
- Response arrives in **< 2000ms** (measure with `curl -w "%{time_total}"`) ✓

---

## Scenario 2 — No Route Error (Edge Case)

**Goal**: Verify clean error when pair has no liquidity.

```bash
# Use a testnet token with no Uniswap/Aerodrome liquidity
curl "http://localhost:3000/v1/quote?tokenIn=0xEeee...&tokenOut=0x000000000000000000000000000000000000dEaD&amountIn=1000000"
```

Expected:

- HTTP `404` ✓
- Body: `{ "error": { "code": "NO_ROUTE", ... } }` ✓

---

## Scenario 3 — Execute Swap: ETH → USDC (US2, SC-002, SC-003, SC-004)

**Goal**: Verify atomic swap executes, fee is deducted, tokens arrive.

1. Open frontend on `http://localhost:5173`
2. Connect MetaMask (Base Sepolia)
3. Enter: `0.1 ETH → USDC`, confirm quote loads
4. Click **Swap**, approve tx in MetaMask
5. Wait for confirmation

Verify on-chain (Basescan Sepolia):

- [ ] Tx succeeded (not reverted) ✓
- [ ] User receives USDC in wallet ≥ `bestAmountOutNet` from quote ✓
- [ ] `SwapExecuted` event emitted with non-zero `feeCollected` ✓
- [ ] `AggregatorRouter` contract USDC balance increased by `feeCollected` ✓
- [ ] User ETH balance decreased by `amountIn + gas` only (no extra deduction) ✓

---

## Scenario 4 — Slippage Revert (US2 Acceptance Scenario 2, SC-004)

**Goal**: Verify the contract reverts when slippage is exceeded, funds are safe.

1. Deploy a test contract that manipulates price in the same block (or use a very tight
   slippage setting: `slippageBps=1`)
2. Submit swap tx
3. Force price move before confirmation (using Anvil's impersonation on forked Base)

Verify:

- [ ] Tx reverted ✓
- [ ] User token balance unchanged (minus gas only) ✓
- [ ] No `SwapExecuted` event emitted ✓

---

## Scenario 5 — Health Check (Constitution V)

```bash
curl http://localhost:3000/v1/health
```

Expected:

- HTTP `200` ✓
- `status` is `"ok"` or `"degraded"` (never missing) ✓
- Response in < 100ms ✓

---

## Scenario 6 — Fee Sweep (FR-007)

1. Execute 3 swaps via Scenario 3
2. Call `sweepFees(USDC_ADDRESS)` as contract owner from hardhat/cast:

```bash
cast send $AGGREGATOR_ADDRESS "sweepFees(address)" $USDC_ADDRESS \
  --private-key $OWNER_KEY --rpc-url $BASE_SEPOLIA_RPC
```

Verify:

- [ ] `FeesSwept` event emitted ✓
- [ ] Treasury address USDC balance increased by total fees from the 3 swaps ✓
- [ ] Contract USDC balance is 0 after sweep ✓

---

## Pass Criteria

All 6 scenarios must pass before mainnet deployment is permitted (per Definition of Done).
