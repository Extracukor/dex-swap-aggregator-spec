# Data Model: Core Swap Aggregation

**Feature**: `001-core-swap-aggregation`
**Date**: 2026-04-02

---

## On-Chain: Smart Contract State & Events

### `AggregatorRouter` Contract

**Storage**:

```text
owner: address                 // Ownable — hardware wallet / multisig
paused: bool                   // Pausable — emergency stop
feeBps: uint16                 // Protocol fee in basis points (default: 5 = 0.05%)
treasury: address              // Accumulated fees are swept here
supportedDexes: mapping(       // Registered DEX adapters
  bytes32 dexId => address adapterAddress
)
```

**Enums**:

```solidity
enum DexId { UNISWAP_V3, AERODROME }
```

**Events**:

```solidity
event SwapExecuted(
    address indexed user,
    address indexed tokenIn,
    address indexed tokenOut,
    uint256 amountIn,
    uint256 amountOut,       // gross output before fee
    uint256 feeCollected,    // in tokenOut
    uint256 amountOutNet,    // amountOut - feeCollected, delivered to user
    bytes32 dexId,           // which DEX routed this swap
    bytes   routePath        // encoded hop path
);

event FeesSwept(
    address indexed token,
    address indexed treasury,
    uint256 amount
);

event FeeUpdated(uint16 oldFeeBps, uint16 newFeeBps);
event Paused(address by);
event Unpaused(address by);
```

---

## Off-Chain: Routing Engine Types (TypeScript)

### `Token`

```typescript
interface Token {
  address: `0x${string}`;   // checksummed EVM address, or sentinel for ETH
  symbol: string;
  decimals: number;
  chainId: number;
  isNative: boolean;        // true for ETH (uses sentinel address)
}
```

### `SwapRoute`

```typescript
interface SwapRoute {
  dexId: 'uniswap_v3' | 'aerodrome';
  path: `0x${string}`[];    // ordered token addresses including intermediates
  fees: number[];           // Uniswap fee tiers per hop (bps*100); empty for Aerodrome
  poolTypes: ('volatile' | 'stable' | null)[];  // Aerodrome pool type per hop; null for Uniswap
  amountOut: bigint;        // raw output from DEX (before protocol fee), in tokenOut decimals
  priceImpactBps: number;   // price impact in basis points (e.g. 50 = 0.5%)
  gasEstimate: bigint;      // estimated gas units
}
```

### `SwapQuote`

```typescript
interface SwapQuote {
  tokenIn: Token;
  tokenOut: Token;
  amountIn: bigint;           // in tokenIn decimals
  bestRoute: SwapRoute;       // highest amountOut after protocolFee
  allRoutes: SwapRoute[];     // all routes, sorted descending by amountOut
  protocolFeeBps: number;     // fee rate at quote time (e.g. 5 = 0.05%)
  protocolFeeAmount: bigint;  // in tokenOut decimals
  bestAmountOutNet: bigint;   // bestRoute.amountOut - protocolFeeAmount
  estimatedGasUSD: number;    // float, informational
  highPriceImpact: boolean;   // true if bestRoute.priceImpactBps > 500
  quoteTimestamp: number;     // Unix epoch ms
  validForSeconds: 30;        // hardcoded per research decision
}
```

### `SwapRequest` (frontend → routing engine)

```typescript
interface SwapRequest {
  tokenIn: `0x${string}`;    // token address or ETH sentinel
  tokenOut: `0x${string}`;
  amountIn: string;          // decimal string (e.g. "100000000" for 100 USDC)
  slippageBps: number;       // user's slippage tolerance (10–500)
  chainId: 8453;             // Base mainnet only for v1
}
```

### `SwapExecution` (off-chain record, written after tx confirmation)

```typescript
interface SwapExecution {
  txHash: `0x${string}`;
  blockNumber: bigint;
  chainId: number;
  userAddress: `0x${string}`;
  tokenIn: `0x${string}`;
  tokenOut: `0x${string}`;
  amountIn: bigint;
  amountOut: bigint;         // gross
  feeCollected: bigint;
  amountOutNet: bigint;
  dexId: string;
  routePath: `0x${string}`[];
  status: 'success' | 'reverted';
  timestamp: number;         // Unix epoch ms of block
}
```

---

## Relationships

```text
SwapRequest (user input)
    │
    ▼
SwapQuote (routing engine output)
    │  contains 1..N
    ▼
SwapRoute (per DEX)
    │  best route selected
    ▼
AggregatorRouter.swap() call  ──► SwapExecuted event (on-chain)
    │
    ▼
SwapExecution (off-chain record, indexed from event log)
```

---

## Allowlist Config (routing engine)

```typescript
// Stored in chain-specific config file, not hardcoded in contract
interface TokenConfig {
  [chainId: number]: Token[];
}

interface ChainConfig {
  chainId: number;
  rpcUrls: string[];          // primary + fallback
  aggregatorAddress: `0x${string}`;
  uniswapV3: {
    quoterV2: `0x${string}`;
    swapRouter02: `0x${string}`;
    feeTiers: number[];       // [100, 500, 3000, 10000]
  };
  aerodrome: {
    router: `0x${string}`;
    factory: `0x${string}`;
  };
  weth: `0x${string}`;
  ethUsdPriceFeed: `0x${string}`;  // Chainlink
}
```
