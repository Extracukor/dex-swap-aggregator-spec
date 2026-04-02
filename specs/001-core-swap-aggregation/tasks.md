# Tasks: Core Swap Aggregation — Quote & Execute

**Feature**: `001-core-swap-aggregation`
**Input**: spec.md, plan.md, data-model.md, contracts/routing-api.md, quickstart.md
**Date**: 2026-04-02

**Organization**: Tasks are grouped by component and user story to enable independent
implementation and validation of each increment.

**Legend**:

- `[P]` — can run in parallel with other `[P]` tasks in the same phase
- `[US1]` / `[US2]` / `[US3]` — maps to user story in spec.md
- `[ ]` → `[x]` when complete

---

## Phase 1: Project Setup (Shared Infrastructure)

**Goal**: Repo structure, tooling, and CI foundations. No business logic yet.
Everything here can run in parallel.

- [ ] T001 [P] Create top-level monorepo dirs: `contracts/`, `engine/`, `frontend/`
- [ ] T002 [P] Init Foundry project in `contracts/` (`forge init --no-git`); configure
  `foundry.toml` with Base mainnet + Base Sepolia RPC env vars and optimizer settings
- [ ] T003 [P] Init Python project in `engine/`: create `pyproject.toml`, `requirements.txt`
  with `fastapi>=0.115`, `uvicorn[standard]>=0.30`, `web3>=7.0`, `pydantic>=2.7`,
  `httpx>=0.27`, `pytest>=8`, `pytest-asyncio`, and `.env.example`
- [ ] T004 [P] Init Vite + React 19 + TypeScript project in `frontend/`
  (`pnpm create vite frontend --template react-ts`); install `wagmi@2`, `viem@2`,
  add shadcn/ui init
- [ ] T005 [P] Create root `.gitignore` covering Foundry artifacts (`out/`, `cache/`),
  Python (`__pycache__/`, `.venv/`, `.env`), and Node (`node_modules/`, `dist/`)
- [ ] T006 [P] Create `railway.toml` in `engine/` with start command
  `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
- [ ] T007 [P] Create `frontend/public/_redirects` for Cloudflare Pages SPA routing
  (`/* /index.html 200`)

**Checkpoint**: `forge build` passes; `uvicorn app.main:app` starts (empty app);
`pnpm dev` in `frontend/` serves the Vite default page.

---

## Phase 2: Foundational — Smart Contract Core (BLOCKING)

**Goal**: `AggregatorRouter` contract fully tested on a Base fork before any
routing engine or frontend work touches it. This phase MUST complete before Phase 3.

### 2a — Contract implementation

- [ ] T010 Create `contracts/src/AggregatorRouter.sol`:
  - `Ownable`, `Pausable`, `ReentrancyGuard` from OpenZeppelin 5.x
  - Storage: `feeBps` (uint16, default 5), `treasury` (address), `paused`
  - `swap(address tokenIn, address tokenOut, uint256 amountIn, uint256 minAmountOut, bytes32 dexId, bytes calldata path)` — external payable
  - On-chain ETH→WETH wrap using `IWETH.deposit{value: amountIn}()`
  - Fee deduction: `feeAmount = amountOut * feeBps / 10000`; fee held in contract
  - `sweepFees(address token)` — owner only, transfers contract token balance to `treasury`
  - `setFeeBps(uint16)` — owner only, max 100 bps guard
  - Emit `SwapExecuted`, `FeesSwept`, `FeeUpdated` events per data-model.md
  - Revert on: paused, zero amountIn, unsupported dexId, output < minAmountOut
- [ ] T011 Create `contracts/src/adapters/UniswapV3Adapter.sol`:
  - Decodes packed path `(address, uint24, address)+`
  - Calls `ISwapRouter02.exactInput()` or `exactInputSingle()` on Base `SwapRouter02`
  - Returns gross `amountOut`
- [ ] T012 Create `contracts/src/adapters/AerodromeAdapter.sol`:
  - Decodes ABI-encoded `(address from, address to, bool stable)[]` route array
  - Calls `IAerodromeRouter.swapExactTokensForTokens()` on Base Aerodrome Router
  - Returns gross `amountOut`
- [ ] T013 [P] Create `contracts/src/interfaces/` — minimal interfaces:
  `IWETH.sol`, `IERC20.sol` (or import OZ), `IUniswapV3SwapRouter.sol`,
  `IAerodromeRouter.sol`

### 2b — Contract tests (write BEFORE running, verify they fail first)

- [ ] T014 Create `contracts/test/unit/FeeCalc.t.sol`:
  - Test `feeBps=5` on various `amountOut` values (integer truncation edge cases)
  - Test `setFeeBps` reverts above 100 bps
  - Test `sweepFees` transfers correct balance to treasury
- [ ] T015 Create `contracts/test/fork/AggregatorRouter.t.sol` — fork Base mainnet:
  - `testSwapETHToUSDC_UniswapV3`: swap 0.1 ETH → USDC via Uniswap v3, verify:
    - User receives USDC ≥ `minAmountOut`
    - `SwapExecuted` emitted with correct fields
    - Contract holds `feeCollected` USDC, user holds net USDC
  - `testSwapETHToUSDC_Aerodrome`: same flow via Aerodrome volatile pool
  - `testSlippageReverts`: set `minAmountOut` above actual output, assert revert
  - `testPauseBlocksSwap`: pause contract, assert `swap()` reverts
  - `testNonOwnerCannotSweep`: assert `sweepFees` reverts for non-owner
- [ ] T016 Create `contracts/test/fork/FeeCollection.t.sol` — fork Base mainnet:
  - Execute 3 swaps, assert cumulative fees match sum of `SwapExecuted.feeCollected`
  - Call `sweepFees`, assert treasury balance increases, contract balance → 0
  - Assert `FeesSwept` event emitted with correct args

### 2c — Deployment script

- [ ] T017 Create `contracts/script/Deploy.s.sol`:
  - Deploy `AggregatorRouter` with constructor args: `feeBps=5`, `treasury=OWNER`
  - Register Uniswap v3 and Aerodrome adapter dexIds
  - Print deployed address to stdout (for `.env` update)

**Checkpoint**: `forge test --fork-url $BASE_RPC_URL` — all tests green.
`forge build` — zero warnings on contracts.

---

## Phase 3: User Story 1 — Get Best Swap Quote (Priority: P1) 🎯 MVP

**Goal**: Routing engine returns a valid quote from ≥2 DEX sources in <2s.
Independent test: `curl` the `/v1/quote` endpoint and inspect the response.

### 3a — Engine foundation

- [ ] T020 Create `engine/app/models.py` — all Pydantic models from data-model.md:
  `Token`, `SwapRoute`, `SwapQuote`, `SwapRequest`, `SwapExecution`
- [ ] T021 Create `engine/app/config/chains.py` — `ChainConfig` dataclass for Base
  mainnet (chainId 8453) and Base Sepolia (84532) with all contract addresses from
  research.md; RPC URLs loaded from env vars
- [ ] T022 Create `engine/app/config/tokens.py` — v1 token allowlist (7 tokens) with
  checksummed addresses from research.md
- [ ] T023 Create `engine/app/main.py` — FastAPI app, include routers, configure
  CORS (allow `*` for v1), structured JSON logging via `python-json-logger`

### 3b — DEX quoters

- [ ] T024 Create `engine/app/quoters/uniswap_v3.py`:
  - Async function `get_quote(w3, chain_config, token_in, token_out, amount_in) -> SwapRoute | None`
  - Call `QuoterV2.quoteExactInput()` via web3.py for all fee tiers [100, 500, 3000, 10000]
  - Try direct pair first; if no liquidity, try 1-hop via WETH
  - Return `SwapRoute` with highest `amount_out`; return `None` if no liquidity
- [ ] T025 Create `engine/app/quoters/aerodrome.py`:
  - Async function `get_quote(w3, chain_config, token_in, token_out, amount_in) -> SwapRoute | None`
  - Call `AerodromeRouter.getAmountsOut()` for both volatile and stable pool types
  - Return `SwapRoute` with highest `amount_out`; return `None` if no liquidity

### 3c — Aggregator and quote route

- [ ] T026 Create `engine/app/aggregator.py`:
  - `async def get_best_quote(request: SwapRequest, chain_config, tokens) -> SwapQuote`
  - Validate `token_in` and `token_out` are in allowlist; raise `TokenNotAllowed` if not
  - Fetch `feeBps` from `AggregatorRouter.getFeeBps()` at startup (cached; refreshed on `FeeUpdated` event)
  - Fetch ETH/USD from Chainlink `latestRoundData()` (cached 60s) for `estimated_gas_usd`
  - Run `uniswap_v3.get_quote()` and `aerodrome.get_quote()` concurrently with `asyncio.gather`
  - Filter `None` results; raise `NoRouteError` if all sources return `None`
  - Sort routes descending by `amount_out`; select best
  - Compute `protocol_fee_amount`, `best_amount_out_net`, `high_price_impact`
  - Return `SwapQuote` with `valid_for_seconds=30`
- [ ] T027 Create `engine/app/routes/quote.py`:
  - `GET /v1/quote` handler; parse query params into `SwapRequest` (Pydantic validation)
  - Call `aggregator.get_best_quote()`
  - Map `NoRouteError` → HTTP 404 `NO_ROUTE`
  - Map validation errors → HTTP 400 with structured error body per API contract
  - Return `SwapQuote` serialized as JSON (snake_case → camelCase via Pydantic alias)
- [ ] T028 [P] Create `engine/app/routes/health.py`:
  - `GET /v1/health` — check RPC connectivity; return `status: ok/degraded`, `uptime`, `version`
- [ ] T029 [P] Create `engine/app/routes/tokens.py`:
  - `GET /v1/tokens` — return allowlist for requested `chainId`

### 3d — Engine tests

- [ ] T030 [P] Create `engine/tests/test_quote.py`:
  - Mock web3.py RPC calls; test `SwapRequest` validation (zero amount, unsupported token,
    slippage out of range)
  - Test `NO_ROUTE` 404 when both quoters return `None`
  - Test correct best-route selection when Uniswap returns higher than Aerodrome
  - Test `high_price_impact=True` when `price_impact_bps > 500`
- [ ] T031 [P] Create `engine/tests/test_health.py`:
  - Test `/v1/health` returns 200 with `status` field present
  - Test `status: degraded` when primary RPC unreachable (mock)

**Checkpoint — US1**: `curl "localhost:3000/v1/quote?tokenIn=0xEeee...&tokenOut=0x8335...&amountIn=100000000000000000"` returns a valid `SwapQuote` JSON with `allRoutes` length ≥ 2 in < 2000ms. Matches Quickstart Scenario 1.

---

## Phase 4: User Story 2 — Execute Swap on Best Route (Priority: P1) 🎯 MVP

**Goal**: Frontend connects wallet, fetches quote, builds and sends swap tx.
Independent test: execute a swap on Base Sepolia, verify `SwapExecuted` event on-chain.

### 4a — Frontend foundation

- [ ] T040 Create `frontend/src/config/wagmi.ts`:
  - Configure wagmi with Base + Base Sepolia chains
  - Connectors: MetaMask, Coinbase Wallet, WalletConnect
  - Transport: `http()` pointing to routing engine RPC proxy (or public Base RPC for frontend)
- [ ] T041 Create `frontend/src/App.tsx`:
  - `WagmiProvider` + `QueryClientProvider` wrappers
  - Render `<SwapWidget />`

### 4b — Quote hook

- [ ] T042 Create `frontend/src/hooks/useQuote.ts`:
  - Accept `tokenIn`, `tokenOut`, `amountIn`, `slippageBps` as inputs
  - Fetch `/v1/quote` via `fetch`; re-fetch every 30 seconds (match `validForSeconds`)
  - Expose: `quote`, `isLoading`, `isExpired`, `error`
  - Set `isExpired=true` when `Date.now() > quoteTimestamp + validForSeconds * 1000`

### 4c — Swap execution hook

- [ ] T043 Create `frontend/src/hooks/useSwap.ts`:
  - Accept `quote: SwapQuote` and `slippageBps: number`
  - Compute `minAmountOut = bestRoute.amountOut * (10000 - slippageBps) / 10000`
  - Encode `path` bytes per DEX (Uniswap v3 packed encoding vs Aerodrome ABI encoding)
  - If `tokenIn` is ERC-20: check allowance; if insufficient, send `approve` tx first
  - Build `AggregatorRouter.swap()` calldata via viem `encodeFunctionData`
  - Send tx via wagmi `useWriteContract`; expose `txHash`, `isPending`, `isSuccess`, `error`

### 4d — Swap UI components

- [ ] T044 Create `frontend/src/components/TokenSelector.tsx`:
  - Fetch allowlist from `/v1/tokens`; render dropdown with token symbol + address
- [ ] T045 Create `frontend/src/components/PriceImpactWarning.tsx`:
  - Render prominent warning banner when `quote.highPriceImpact === true`
- [ ] T046 Create `frontend/src/components/SwapWidget.tsx`:
  - Token pair selector (uses `TokenSelector`)
  - Amount input field
  - Quote display: best output, price impact %, estimated gas USD, protocol fee
  - Slippage setting (default 0.5%, adjustable 0.1%–5%)
  - Quote expiry countdown (30s)
  - Swap button: disabled when no quote / expired / wallet not connected
  - Loading states and error messages
  - On success: show tx hash with Basescan link

**Checkpoint — US2**: Connect MetaMask to Base Sepolia, enter 0.01 ETH → USDC, swap executes, `SwapExecuted` event visible on Basescan Sepolia. Matches Quickstart Scenarios 3 and 4.

---

## Phase 5: User Story 3 — Compare All Routes (Priority: P2)

**Goal**: Power-users can see all DEX routes ranked side by side and select manually.
Independent test: after quote loads, expand route comparison view and verify all routes visible.

- [ ] T050 Create `frontend/src/components/RouteComparison.tsx`:
  - Collapsible "Show all routes" section below the main quote
  - Render each `SwapRoute` in `allRoutes`: DEX name, output amount, price impact, estimated gas
  - Highlight best route; allow user to click and select an alternative
- [ ] T051 Update `frontend/src/hooks/useSwap.ts` to accept an optional `selectedRoute`
  override; use it instead of `quote.bestRoute` when set
- [ ] T052 Update `frontend/src/components/SwapWidget.tsx` to render `<RouteComparison />`
  and wire the selected route into `useSwap`

**Checkpoint — US3**: After loading a quote, expand "Show all routes", see ≥2 routes ranked, click the non-best route, observe swap button uses the selected DEX. Matches Quickstart Scenario 1 (allRoutes length check).

---

## Phase 6: Polish & Cross-Cutting Concerns

**Goal**: Production readiness before testnet sign-off.

- [ ] T060 [P] Add `SwapExecution` off-chain recording in `engine/`:
  - Subscribe to `SwapExecuted` events via web3.py filter on startup
  - Append each event to a local JSONL file (`data/executions.jsonl`) for revenue tracking
  - This satisfies Constitution V (swap events mirrored off-chain)
- [ ] T061 [P] Add alert logic in `engine/app/main.py`:
  - Track swap success/failure counts in a rolling 1-hour window (in-memory)
  - Log a `CRITICAL` structured log entry when success rate drops below 95%
  - Expose current success rate via `/v1/health` response
- [ ] T062 [P] Add `Content-Security-Policy` and `X-Frame-Options` headers to FastAPI
  app (security hardening for the API)
- [ ] T063 [P] Add `frontend/src/components/NetworkGuard.tsx`:
  - Detect if wallet is connected to wrong chain; prompt to switch to Base
- [ ] T064 [P] Run `forge test --fork-url $BASE_RPC_URL` final pass, fix any failures
- [ ] T065 [P] Run full Quickstart validation (all 6 scenarios) on Base Sepolia
- [ ] T066 Write `contracts/README.md` with deploy instructions (env vars, `forge script` command)
- [ ] T067 Write `engine/README.md` with local dev and Railway deploy instructions
- [ ] T068 Write `frontend/README.md` with local dev and Cloudflare Pages deploy instructions

**Checkpoint — Definition of Done**: All 6 Quickstart scenarios pass on Base Sepolia.
Fork tests green. Revenue monitoring active (T060 + T061). Ready for independent code review.

---

## Dependencies & Execution Order

### Phase Dependencies

```text
Phase 1 (Setup)        → no dependencies, start immediately
Phase 2 (Contracts)    → requires Phase 1 complete — BLOCKS Phases 3–5
Phase 3 (US1 Quote)    → requires Phase 2 complete (for contract address + ABI)
Phase 4 (US2 Execute)  → requires Phase 3 complete (quote hook needed)
Phase 5 (US3 Routes)   → requires Phase 3 complete; can overlap with Phase 4
Phase 6 (Polish)       → requires Phases 3–5 complete
```

### Within Each Phase

- All `[P]` tasks within a phase can be worked simultaneously
- Non-`[P]` tasks must be done in listed order within their phase

### MVP Scope (minimum to validate revenue model)

Complete through Phase 4 only (US1 + US2). This is sufficient to:

1. Accept a token pair and amount
2. Return best quote from 2 DEX sources
3. Execute a swap with on-chain fee collection

Phase 5 (US3) and Phase 6 (Polish) are enhancements — not blockers for first testnet validation.

---

## Implementation Strategy

### Solo Developer — Sequential Priority Order

1. Phase 1 — Setup (~1 day)
2. Phase 2 — Contracts + fork tests (~3–4 days)
3. Phase 3 — Routing engine + quote API (~3 days)
4. Phase 4 — Frontend + swap execution (~3 days)
5. Phase 5 — Route comparison (~1 day)
6. Phase 6 — Polish + sign-off (~1–2 days)

**Total estimate**: ~12–14 development days to testnet sign-off.

---

## Notes

- `[P]` tasks have no file conflicts and no output dependencies within their phase
- After T015/T016 (fork tests), commit before writing any adapter code — ensures a clean baseline
- Never skip the `minAmountOut` validation path — it is the primary user fund safety mechanism
- JSONL execution log (T060) is the v1 revenue tracking solution; a proper DB/dashboard is a future spec
- `feeBps` is read from the contract at engine startup — do not hardcode in the engine
