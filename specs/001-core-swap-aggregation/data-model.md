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

## Off-Chain: Routing Engine Models (Python / Pydantic v2)

All models live in `engine/app/models.py`.
`int` fields representing on-chain `uint256` amounts use Python's native arbitrary-precision `int`.

### `Token`

```python
from pydantic import BaseModel

class Token(BaseModel):
    address: str        # checksummed EVM address, or ETH sentinel "0xEeee...eEEE"
    symbol: str
    decimals: int
    chain_id: int
    is_native: bool     # True for ETH (sentinel address)
```

### `SwapRoute`

```python
from typing import Literal

class SwapRoute(BaseModel):
    dex_id: Literal["uniswap_v3", "aerodrome"]
    path: list[str]             # ordered checksummed token addresses incl. intermediates
    fees: list[int]             # Uniswap fee tiers per hop (e.g. 500); empty for Aerodrome
    pool_types: list[Literal["volatile", "stable"] | None]
                                # Aerodrome pool type per hop; None for Uniswap hops
    amount_out: int             # raw DEX output (before protocol fee), in tokenOut decimals
    price_impact_bps: int       # price impact in basis points (e.g. 50 = 0.5%)
    gas_estimate: int           # estimated gas units
```

### `SwapQuote`

```python
class SwapQuote(BaseModel):
    token_in: Token
    token_out: Token
    amount_in: int              # in tokenIn decimals
    best_route: SwapRoute       # route with highest amount_out after protocol fee
    all_routes: list[SwapRoute] # sorted descending by amount_out
    protocol_fee_bps: int       # fee rate at quote time (e.g. 5 = 0.05%)
    protocol_fee_amount: int    # in tokenOut decimals
    best_amount_out_net: int    # best_route.amount_out - protocol_fee_amount
    estimated_gas_usd: float    # informational display only
    high_price_impact: bool     # True if best_route.price_impact_bps > 500
    quote_timestamp: int        # Unix epoch milliseconds
    valid_for_seconds: int = 30
```

### `SwapRequest` (query params, frontend → routing engine)

```python
from pydantic import Field

class SwapRequest(BaseModel):
    token_in: str               # EVM address or ETH sentinel
    token_out: str
    amount_in: str              # decimal integer string (e.g. "100000000" for 100 USDC)
    slippage_bps: int = Field(default=50, ge=10, le=500)
    chain_id: int = 8453        # Base mainnet only for v1
```

### `SwapExecution` (off-chain record, written after tx confirmation)

```python
from typing import Literal

class SwapExecution(BaseModel):
    tx_hash: str
    block_number: int
    chain_id: int
    user_address: str
    token_in: str
    token_out: str
    amount_in: int
    amount_out: int             # gross output
    fee_collected: int
    amount_out_net: int
    dex_id: str
    route_path: list[str]
    status: Literal["success", "reverted"]
    timestamp: int              # Unix epoch ms of block
```

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

Stored in `engine/app/config/chains.py` and `engine/app/config/tokens.py`.

```python
from dataclasses import dataclass, field

@dataclass
class UniswapV3Config:
    quoter_v2: str
    swap_router_02: str
    fee_tiers: list[int] = field(default_factory=lambda: [100, 500, 3000, 10000])

@dataclass
class AerodromeConfig:
    router: str
    factory: str

@dataclass
class ChainConfig:
    chain_id: int
    rpc_urls: list[str]             # index 0 = primary, index 1+ = fallbacks
    aggregator_address: str
    uniswap_v3: UniswapV3Config
    aerodrome: AerodromeConfig
    weth: str
    eth_usd_price_feed: str         # Chainlink ETH/USD on Base

# Token allowlist: list[Token] per chain_id
# Defined in engine/app/config/tokens.py
```
