# Tasks: Frontend PoC Deployment

**Feature**: `003-frontend-poc-deployment`
**Input**: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/mock-service-contract.md`, `quickstart.md`
**Date**: 2026-04-05

**Legend**:

- `[P]` = párhuzamosan végezhető ugyanazon fázison belül
- `[US1]` / `[US2]` / `[US3]` = user story hozzárendelés
- `[FR-xxx]` = követelmény hozzárendelés
- `[ ]` -> `[x]` készültség jelölése

---

## Phase 1: Project Setup (Shared Foundation)

**Goal**: TypeScript-only frontend projekt bootstrap, wallet stack, Tailwind/shadcn és alap konfigok.

- [ ] T001 [P] Hozd létre az `apps/frontend-poc/` mappát
- [ ] T002 [P] Futtasd: `npm create vite@latest . -- --template react-ts` az `apps/frontend-poc/` könyvtárban [FR-001]
- [ ] T003 [P] Futtasd: `npm install`
- [ ] T004 [P] Ellenőrizd, hogy a scaffold TypeScript (`.ts`/`.tsx`) fájlokat használ [FR-002]
- [ ] T005 [P] Telepítsd a wallet stack-et: `wagmi`, `viem`, `@rainbow-me/rainbowkit`, `@tanstack/react-query` [FR-003]
- [ ] T006 [P] Ellenőrizd, hogy `ethers` és `web3modal` nincs installálva (`npm ls ethers web3modal`) [FR-003]
- [ ] T007 [P] Telepítsd Tailwind dependency-ket: `tailwindcss`, `postcss`, `autoprefixer` [FR-004]
- [ ] T008 [P] Futtasd: `npx tailwindcss init -p` [FR-004]
- [ ] T009 [P] Nevezd át `tailwind.config.js` -> `tailwind.config.ts` [FR-004]
- [ ] T010 [P] Hozd létre/konvertáld a PostCSS konfigurációt TypeScript-re (`postcss.config.ts`), ha a toolchain támogatja; ellenkező esetben dokumentáld a kivételt [FR-004]
- [ ] T011 [P] Add hozzá Tailwind direktívákat a `src/index.css` fájlban: `@tailwind base;`, `@tailwind components;`, `@tailwind utilities;` [FR-004]
- [ ] T012 [P] Futtasd: `npx shadcn@latest init` [FR-005]
- [ ] T013 [P] Futtasd: `npx shadcn@latest add button select card skeleton` [FR-005]
- [ ] T014 [P] Telepítsd a `sonner` csomagot [FR-005]
- [ ] T015 [P] Hozd létre a `wrangler.toml` fájlt `pages_build_output_dir = "dist"` értékkel [FR-009]
- [ ] T016 [P] Ellenőrizd, hogy `npm run build` sikeresen előállítja a `dist/` könyvtárat [FR-009]

**Checkpoint**: A projekt buildel, TypeScript-only forrásokkal, wallet stack helyes, Tailwind + shadcn működik, Cloudflare output config kész.

---

## Phase 2: Core App Skeleton and Providers

**Goal**: App shell és provider tree a jó runtime rétegezéssel.

- [ ] T020 Készítsd el a `src/config/chains.ts` fájlt `baseSepolia` exporttal [FR-007]
- [ ] T021 Készítsd el a `src/config/wagmi.ts` fájlt úgy, hogy csak `baseSepolia` legyen támogatott chain [FR-007]
- [ ] T022 Készítsd el a `src/config/tokens.ts` curated token listát (`TokenOption[]`) [US1]
- [ ] T023 Készítsd el a `src/types/quote.ts` típusokat: `QuoteRequest`, `QuoteResult`, `QuoteState`, `TokenOption` [FR-006]
- [ ] T024 Készítsd el a `src/components/layout/AppShell.tsx` alap layoutot
- [ ] T025 Készítsd el a `src/components/wallet/WalletConnectButton.tsx` komponenst RainbowKit kapcsolódással [US1]
- [ ] T026 Készítsd el a `src/components/wallet/NetworkGuardBanner.tsx` komponenst [US2, FR-008]
- [ ] T027 Állítsd be a `QueryClientProvider`, `WagmiProvider`, `RainbowKitProvider` provider tree-t a `src/main.tsx` fájlban [US1]
- [ ] T028 Integráld az `AppShell` + wallet komponenseket az `src/App.tsx` fájlba

**Checkpoint**: Az app indul, wallet connect gomb látszik, provider lánc helyes.

---

## Phase 3: UI Components (Swap Flow)

**Goal**: shadcn-alapú swap UI komponensek és validációs viselkedés.

- [ ] T030 [US1] Készítsd el a `src/components/swap/TokenSelect.tsx` komponenst shadcn `Select`-tel [FR-005]
- [ ] T031 [US1] Készítsd el a `src/components/swap/AmountInput.tsx` komponenst numerikus validációval
- [ ] T032 [US1] Készítsd el a `src/components/swap/QuoteButton.tsx` komponenst shadcn `Button`-nel [FR-005]
- [ ] T033 [US1] Készítsd el a `src/components/swap/QuoteResultCard.tsx` komponenst shadcn `Card` + `Skeleton` használattal [FR-005]
- [ ] T034 [US1] Készítsd el a `src/components/swap/SwapWidget.tsx` komponenst a teljes input+state orchestrationnel
- [ ] T035 [US2] Implementáld a "same token" tiltást (`tokenIn !== tokenOut`) [FR-008]
- [ ] T036 [US2] Implementáld az amount validációt (`amountIn > 0`) és a tiltott állapotot [FR-008]
- [ ] T037 [US2] Implementáld az inline validációs üzeneteket:
  - `tokenIn and tokenOut must be different`
  - `Enter a valid amount`
- [ ] T038 [US2] Kösd össze a `NetworkGuardBanner` állapotát a form disable logikával [FR-008]
- [ ] T039 [US2] Add hozzá a `useSwitchChain` alapú chain-váltás CTA-t a bannerbe [FR-008]

**Checkpoint**: A swap UI teljesen renderel, validációk működnek, rossz chainen a form tiltott.

---

## Phase 4: Mock Quote Service and Hook Integration

**Goal**: 1 másodperces mock quote flow és pontos output contract.

- [ ] T040 [US1] Készítsd el a `src/services/MockQuoteService.ts` fájlt [FR-006]
- [ ] T041 [US1] Definiáld a bemenetet pontosan: `{ tokenIn: string; tokenOut: string; amountIn: string }` [FR-006]
- [ ] T042 [US1] Implementáld az explicit 1 másodperces delay-t: `await new Promise(resolve => setTimeout(resolve, 1000))` [FR-006]
- [ ] T043 [US1] Add vissza pontosan ezt az objektumot: `{ outputAmount, priceImpact, protocolFee }` [FR-006]
- [ ] T044 [US1] Gondoskodj róla, hogy ne legyen extra mező a visszatérési objektumban [FR-006]
- [ ] T045 [US1] Készítsd el a `src/hooks/useQuote.ts` hookot `QuoteState` menedzsmenttel
- [ ] T046 [US1] Kösd össze a `SwapWidget`-et a `useQuote` hookkal és a `MockQuoteService`-szel
- [ ] T047 [US1] Implementáld, hogy input változásra a korábbi quote törlődjön és új 1s ciklus induljon
- [ ] T048 [US1] Implementáld a loading state megjelenítést (`Skeleton`) a mock kérés alatt
- [ ] T049 [US1] Integráld a `sonner` toast hibakezelést mock hiba esetére [FR-005]

**Checkpoint**: A quote flow végponttól végpontig működik, 1 másodperces töltéssel és megfelelő mezőkkel.

---

## Phase 5: Cloudflare Pages Deployment

**Goal**: PoC publikus URL-en elérhető Cloudflare Pages alatt.

- [ ] T050 [US3] Ellenőrizd a `wrangler.toml` tartalmat:
  - `name`
  - `compatibility_date`
  - `pages_build_output_dir = "dist"` [FR-009]
- [ ] T051 [US3] Futtasd a lokális buildet: `npm run build`
- [ ] T052 [US3] Deployold manuálisan: `npx wrangler pages deploy dist`
- [ ] T053 [US3] Verifikáld, hogy a deploy URL HTTP 200-at ad [SC-002]
- [ ] T054 [US3] Verifikáld, hogy a Connect Wallet gomb látható a deployed appban
- [ ] T055 [US3] Verifikáld, hogy Tailwind/shadcn stílusok helyesen jelennek meg
- [ ] T056 [US3] Futtasd le élő URL-en az US1 és US2 acceptance scenario-kat

**Checkpoint**: Cloudflare Pages deploy sikeres, élő demó használható.

---

## Phase 6: Validation, Safety, and Regression Checks

**Goal**: FR és SC megfelelés lezárása.

- [ ] T060 [P] Ellenőrizd, hogy `src/` alatt nincs `.js`/`.jsx` forrás [SC-003]
- [ ] T061 [P] Ellenőrizd, hogy nincs `ethers`, `web3modal`, wagmi v1 referencia a `package.json`-ban [SC-004]
- [ ] T062 [P] Írj unit tesztet a `MockQuoteService` időzítésre (>=1000ms, <=1200ms) [SC-001]
- [ ] T063 [P] Írj unit tesztet a `MockQuoteService` output shape-re (`outputAmount`, `priceImpact`, `protocolFee`) [SC-001]
- [ ] T064 [P] Írj tesztet a network guard reakcióidejére (cél <=200ms) [SC-005]
- [ ] T065 [P] Ellenőrizd, hogy nincs `writeContract`/`sendTransaction` útvonal a quote PoC flowban [FR-010]
- [ ] T066 [P] Futtasd végig a `quickstart.md` összes szcenárióját
- [ ] T067 [P] Dokumentáld a validáció eredményeit rövid release note-ban

**Checkpoint**: FR-001..FR-010 és SC-001..SC-006 lezárt, PoC kész.

---

## Dependencies and Order

1. Phase 1 -> kötelező alap minden más előtt.
2. Phase 2 -> provider/config készül el a UI előtt.
3. Phase 3 -> UI komponensek alapjai.
4. Phase 4 -> mock service + quote hook integráció (US1/US2).
5. Phase 5 -> deploy (US3).
6. Phase 6 -> teljes megfelelőségi zárás.

---

## Minimum Deliverable (MVP for this PoC)

A feature minimálisan késznek tekinthető, ha teljesül:

- Phase 1 teljes
- Phase 2 teljes
- Phase 3 teljes
- Phase 4 teljes
- Phase 5 teljes
- Phase 6-ból legalább T060-T066 teljes
