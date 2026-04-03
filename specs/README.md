# specs

Feature specifications are stored here.

Each feature gets its own folder: `[###-feature-name]/`

```text
specs/
└── 001-example-feature/
    ├── spec.md          ← feature specification (user stories, requirements, success criteria)
    ├── plan.md          ← technical implementation plan
    ├── research.md      ← technology research and decisions
    ├── data-model.md    ← entity definitions and relationships
    ├── contracts/       ← API / interface contracts
    ├── quickstart.md    ← key validation scenarios
    └── tasks.md         ← executable task list
```

---

## Feature állapot — 2026-04-03

### 001 — Core Swap Aggregation

**Branch**: `001-core-swap-aggregation` | **Prioritás**: MVP

| Artifact | Állapot | Tartalom |
|----------|---------|----------|
| `spec.md` | ✅ Kész | 3 user story (US1–US3), FR-001–FR-010, SC-001–SC-006 |
| `research.md` | ✅ Kész | 7 döntés: aggregator pattern, Uniswap v3, Aerodrome, FastAPI, hosting |
| `data-model.md` | ✅ Kész | Pydantic v2 modellek (SwapQuote, SwapRoute, stb.), Solidity events |
| `contracts/routing-api.md` | ✅ Kész | REST API contract (GET /v1/quote, /tokens, /health) + Solidity ABI |
| `quickstart.md` | ✅ Kész | 6 testnet validációs scenario (Definition of Done gate) |
| `plan.md` | ✅ Kész | Constitution Check passed (mind az 5 elv ✅), projekt struktúra |
| `tasks.md` | ✅ Kész | 68 task, 6 fázis — Phase 1 (Setup) → Phase 6 (Polish) |
| Implementáció | ⏳ Nem kezdődött | Implementációs repo: github.com/Extracukor/dex-swap-aggregator |

**Következő lépés**: Nyisd meg a [dex-swap-aggregator](https://github.com/Extracukor/dex-swap-aggregator)
repót és kezdd a Phase 1 (Setup) taskok végrehajtásával.

---

### 002 — Automated Growth Engine

**Branch**: `002-automated-growth-engine` | **Prioritás**: Post-MVP

| Artifact | Állapot | Tartalom |
|----------|---------|----------|
| `spec.md` | ✅ Kész | 3 user story: napi stats, DeFiLlama listing, Dune dashboard |
| `plan.md` | ❌ Hiányzik | Létrehozás: `/speckit.plan` — 001 implementáció után |
| `tasks.md` | ❌ Hiányzik | Plan után: `/speckit.tasks` |
| Implementáció | ❌ | 001 után |
