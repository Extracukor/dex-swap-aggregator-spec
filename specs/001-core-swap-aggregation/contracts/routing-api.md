# API Contract: Routing Engine REST API v1

**Feature**: `001-core-swap-aggregation`
**Base URL**: `https://api.yourprotocol.xyz/v1`
**Date**: 2026-04-02

> **Constitution IV**: This contract MUST be finalized before any implementation code
> is written. Breaking changes require a version bump (`/v2`).

---

## Endpoints

### `GET /quote`

Returns the best swap quote and all available routes.

**Query Parameters**:

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `tokenIn` | `string` | Yes | EVM address of input token, or `0xEeee...eEEE` for native ETH |
| `tokenOut` | `string` | Yes | EVM address of output token |
| `amountIn` | `string` | Yes | Input amount as decimal integer string (e.g. `"100000000"` for 100 USDC) |
| `slippageBps` | `integer` | No | Slippage tolerance in bps. Default: `50` (0.5%). Range: `10`–`500` |
| `chainId` | `integer` | No | Default: `8453` (Base). Only `8453` supported in v1 |

**Success Response** `200 OK`:

```json
{
  "tokenIn": {
    "address": "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
    "symbol": "ETH",
    "decimals": 18,
    "isNative": true
  },
  "tokenOut": {
    "address": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "symbol": "USDC",
    "decimals": 6,
    "isNative": false
  },
  "amountIn": "100000000000000000",
  "bestRoute": {
    "dexId": "uniswap_v3",
    "path": [
      "0x4200000000000000000000000000000000000006",
      "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"
    ],
    "fees": [500],
    "poolTypes": [null],
    "amountOut": "248750000",
    "priceImpactBps": 3,
    "gasEstimate": "145000"
  },
  "allRoutes": [
    {
      "dexId": "uniswap_v3",
      "path": ["0x4200...0006", "0x8335...2913"],
      "fees": [500],
      "poolTypes": [null],
      "amountOut": "248750000",
      "priceImpactBps": 3,
      "gasEstimate": "145000"
    },
    {
      "dexId": "aerodrome",
      "path": ["0x4200...0006", "0x8335...2913"],
      "fees": [],
      "poolTypes": ["volatile"],
      "amountOut": "248100000",
      "priceImpactBps": 5,
      "gasEstimate": "130000"
    }
  ],
  "protocolFeeBps": 5,
  "protocolFeeAmount": "124375",
  "bestAmountOutNet": "248625625",
  "estimatedGasUSD": 0.04,
  "highPriceImpact": false,
  "quoteTimestamp": 1743590400000,
  "validForSeconds": 30
}
```

**Error Responses**:

| HTTP | `code` | Description |
| ---- | ------ | ----------- |
| `400` | `INVALID_TOKEN` | tokenIn or tokenOut not in allowlist |
| `400` | `INVALID_AMOUNT` | amountIn is zero, negative, or not a valid integer string |
| `400` | `UNSUPPORTED_CHAIN` | chainId not supported |
| `400` | `SLIPPAGE_OUT_OF_RANGE` | slippageBps outside 10–500 |
| `404` | `NO_ROUTE` | No liquidity found for this pair on any DEX source |
| `503` | `RPC_UNAVAILABLE` | All RPC providers unreachable; retry after delay |

**Error body**:

```json
{
  "error": {
    "code": "NO_ROUTE",
    "message": "No liquidity found for the requested token pair on Base.",
    "retryable": false
  }
}
```

---

### `GET /tokens`

Returns the supported token allowlist for a given chain.

**Query Parameters**:

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `chainId` | `integer` | No | Default: `8453` |

**Success Response** `200 OK`:

```json
{
  "chainId": 8453,
  "tokens": [
    { "address": "0xEeee...eEEE", "symbol": "ETH", "decimals": 18, "isNative": true },
    { "address": "0x4200...0006", "symbol": "WETH", "decimals": 18, "isNative": false },
    { "address": "0x8335...2913", "symbol": "USDC", "decimals": 6, "isNative": false }
  ]
}
```

---

### `GET /health`

Health check endpoint (required by constitution Principle V).

**Success Response** `200 OK`:

```json
{
  "status": "ok",
  "rpc": "ok",
  "uptime": 3601,
  "version": "1.0.0"
}
```

**Degraded Response** `200 OK` (primary RPC down, fallback active):

```json
{
  "status": "degraded",
  "rpc": "fallback",
  "uptime": 3601,
  "version": "1.0.0"
}
```

---

## Smart Contract Interface

### `AggregatorRouter` — `swap()`

Called by the frontend after the user approves token spend.
For ETH swaps: called with `msg.value = amountIn`, no prior approval needed.

```solidity
function swap(
    address tokenIn,          // ERC-20 address, or address(0) for native ETH
    address tokenOut,         // ERC-20 address
    uint256 amountIn,         // exact input amount in tokenIn decimals
    uint256 minAmountOut,     // slippage-adjusted minimum output (from quote minus slippage)
    bytes32 dexId,            // keccak256 of "uniswap_v3" or "aerodrome"
    bytes calldata path       // ABI-encoded routing path (DEX-specific encoding)
) external payable returns (uint256 amountOutNet);
```

**Returns**: net output amount (after protocol fee deduction) sent to `msg.sender`.

**Reverts**:

| Revert reason | Condition |
| ------------- | --------- |
| `Paused()` | Contract is paused |
| `UnsupportedDex()` | dexId not registered |
| `ZeroAmount()` | amountIn is zero |
| `InsufficientOutput(actual, minRequired)` | DEX output < minAmountOut |
| `TokenNotAllowed()` | tokenIn or tokenOut not in on-chain allowlist |

---

### `AggregatorRouter` — `getFeeBps()`

```solidity
function getFeeBps() external view returns (uint16);
```

Returns the current protocol fee in basis points. Called by routing engine at startup
and cached; refreshed if `FeeUpdated` event is detected.

---

## Encoding Conventions

**Uniswap v3 `path`**:
ABI-encoded as `abi.encodePacked(tokenA, fee, tokenB)` per hop.
Example 1-hop: `abi.encodePacked(WETH, uint24(500), USDC)`

**Aerodrome `path`**:
ABI-encoded as array of `(address from, address to, bool stable)` structs per hop.
Example 1-hop: `abi.encode([(WETH, USDC, false)])`

The aggregator contract decodes these internally via the registered DEX adapter.

---

## Versioning Policy

- This is `v1`. All endpoints are prefixed `/v1/`.
- Any breaking change to request/response schema requires a `/v2/` prefix and a
  migration guide committed to this spec directory.
- Additive fields (new optional response fields) are non-breaking and do not require
  a version bump.
