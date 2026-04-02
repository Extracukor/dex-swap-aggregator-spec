# Research: Core Swap Aggregation

**Feature**: `001-core-swap-aggregation`
**Date**: 2026-04-02

---

## Architecture Decision: Aggregator Pattern

**Decision**: "Aggregator-router" hybrid — off-chain routing engine chooses the best
DEX, on-chain contract executes a single atomic swap via a delegate-call or direct
DEX adapter call.

**Rationale**: Pure on-chain routing (like 1inch's on-chain pathfinder) is gas-heavy
and complex. Pure off-chain execution loses atomicity. The hybrid keeps gas costs low
(one tx, one DEX call) while the off-chain engine does the heavy comparison work.

**Alternatives Considered**:

- Full on-chain aggregation (1inch v5 style): Too complex for a solo developer to audit
  safely; very high gas overhead for multi-path splits.
- Direct DEX integration without aggregation: Leaves money on the table; defeats the
  product's core value proposition.

---

## DEX Integration: Uniswap v3 on Base

**Decision**: Use Uniswap v3's `Quoter V2` contract for off-chain quotes and
`SwapRouter02` for on-chain execution via `exactInputSingle` / `exactInput`.

**Rationale**: Uniswap v3 is the largest DEX by TVL on Base. `QuoterV2` provides
gas-accurate simulation quotes. `SwapRouter02` is battle-tested and audited.
The aggregator contract calls `SwapRouter02` directly — no custom swap logic needed
for this DEX.

**Key Base Mainnet Addresses**:

- `SwapRouter02`: `0x2626664c2603336E57B271c5C0b26F421741e481`
- `QuoterV2`: `0x3d4e44Eb1374240CE5F1B136041F4865E9F6F7e7`
- `WETH`: `0x4200000000000000000000000000000000000006`

**Alternatives Considered**:

- Uniswap v4 (Base): Still early; limited liquidity; hook ecosystem immature. Revisit in 6 months.
- Uniswap v2 (base forks): Lower capital efficiency; not all pairs available.

---

## DEX Integration: Aerodrome

**Decision**: Aerodrome is the dominant ve(3,3) DEX on Base by TVL and volume. Use
Aerodrome's `Router` for quotes (`getAmountsOut`) and swap execution. Aerodrome
supports both volatile and stable pools — include both pool types in routing.

**Rationale**: Aerodrome frequently offers better rates than Uniswap v3 for stablecoin
pairs (USDC/USDT, DAI/USDC) due to its stable pool curve. Covering it captures a
large portion of Base liquidity.

**Key Base Mainnet Addresses**:

- `Router`: `0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43`
- `Factory`: `0x420DD381b31aEf6683db6B902084cB0FFECe40Da`

**Alternatives Considered**:

- BaseSwap, SwapBased: Much smaller TVL; adds integration complexity for marginal gain.
  Add in a future spec when volume warrants.

---

## Smart Contract Architecture

**Decision**: Single `AggregatorRouter` contract owned by deployer (multisig).
The contract receives ETH or ERC-20 tokens, wraps ETH if needed, calls the target
DEX router with a minimum output amount, collects fee from output, sends remainder
to user. Pausable via OpenZeppelin `Pausable`. No upgradeable proxy.

**Rationale**: Minimal attack surface. One contract, one code path, easy to audit.
Immutable after deployment reduces governance attack risk. Fee is taken post-swap from
output (not input) to simplify accounting — user always sees net output in quote.

**Fee collection**: On successful swap, `feeAmount = amountOut * feeBps / 10000`.
Fee stays in the contract and is swept by owner periodically to treasury. Not
transferred per-tx to save gas.

**Libraries used**:

- OpenZeppelin 5.x: `Ownable`, `Pausable`, `ReentrancyGuard`, `SafeERC20`
- No custom math; rely on DEX router's output amounts

**Alternatives Considered**:

- Upgradeable proxy: Adds complexity, increases audit scope, introduces governance risk.
  Rejected per constitution Principle I.
- Fee in input token: Complicates allowance math and user UX. Rejected.
- Per-tx treasury transfer: Costs ~21k gas per swap. Batch sweep is more gas-efficient.

---

## Routing Engine: Technology

**Decision**: Node.js 22 LTS + TypeScript 5, HTTP REST API served by Fastify.
Single-process; no message queue needed for v1.

**Rationale**: Matches constitution constraints. Fastify is fast (~70k req/s), low
overhead, JSON schema validation built-in. Single process is sufficient for solo ops;
horizontal scalability can be added later behind a load balancer.

**RPC Strategy**: Primary — Alchemy (Base). Fallback — Ankr public endpoint.
Use `viem` with a fallback transport (`fallback([http(alchemy), http(ankr)])`) to
satisfy the constitution's "no single third-party RPC" constraint.

**Alternatives Considered**:

- Express: Slower than Fastify; no built-in schema validation.
- GraphQL: Over-engineered for a simple quote/execute API.
- Dedicated quote cache (Redis): Premature for v1; quotes are cheap to re-fetch.

---

## Frontend Technology

**Decision**: React 19 + Vite + TypeScript. Wallet connection: wagmi v2 + viem v2
(matches constitution). UI: shadcn/ui components (MIT license, copy-paste style —
no bundled runtime dependency). No Redux; React Query for server state.

**Rationale**: Minimal deps, all MIT licensed. React Query handles quote polling and
cache invalidation cleanly. shadcn/ui gives a professional UI without a heavy library.

**Alternatives Considered**:

- Next.js: SSR unnecessary for a client-side DApp; adds build complexity.
- Chakra UI / MUI: Heavier bundle; more dependencies to audit for license compliance.

---

## Quote Validity & Staleness

**Decision**: Quotes are valid for **30 seconds** (aligned with spec SC-002).
The routing engine returns `quoteTimestamp` + `validForSeconds: 30` in every response.
Frontend displays a countdown and auto-refreshes at expiry.

**Rationale**: Block time on Base is ~2s. 30s allows ~15 blocks worst case.
Slippage protection on-chain is the safety net for price movement within that window.

---

## Token Allowlist (v1)

**Decision**: Hardcoded in routing engine config. Initial tokens on Base:

| Symbol | Address |
| ------ | ------- |
| ETH (native) | `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE` (sentinel) |
| WETH | `0x4200000000000000000000000000000000000006` |
| USDC | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| USDT | `0xfde4C96c8593536E31F229EA8f37b2ADa2699bb2` |
| DAI | `0x50c5725949A6F0c72E6C4a641F24049A917DB0Cb` |
| WBTC | `0x0555E30da8f98308EdB960aa94C0Db47230d2B9` |
| cbETH | `0x2Ae3F1Ec7F1F5012CFEab0185bfc7aa3cf0DEc22` |

**Rationale**: Limiting to known, liquid tokens prevents routing failures, protects
users from fee-on-transfer / rebase token edge cases, and reduces contract audit scope.

---

## Gas Estimation

**Decision**: Off-chain estimate using `viem`'s `estimateGas` against a forked Base RPC
per quote. Show in USD using a cached ETH/USD price (refreshed every 60s from a
public price feed — Chainlink Base ETH/USD: `0x71041dddad3595F9CEd3DcCFBe3D1F4b0a16Bb70`).

**Rationale**: Accurate enough for display; does not affect routing (v1 assumption from spec).
Chainlink is already audited and trusted; no additional oracle dependency needed.
