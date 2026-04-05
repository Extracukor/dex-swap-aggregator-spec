# DEX Swap Aggregator — Spec

Ez a repository a projekt **Spec-Driven Development (SDD)** specifikációit tárolja,
a [github/spec-kit](https://github.com/github/spec-kit) módszertana alapján.

**Implementációs repo**: https://github.com/Extracukor/dex-swap-aggregator

---

## Jelenlegi állapot — 2026-04-05

### Constitution

| Verzió | Ratified | Állapot |
|--------|----------|---------|
| v1.4.0 | 2026-04-02 | ✅ Elfogadva |

### Project Status

- Phase 1: Frontend PoC in progress.

### Feature-ök

| # | Branch | Spec | Plan | Tasks | Implementáció |
|---|--------|------|------|-------|---------------|
| 001 | `001-core-swap-aggregation` | ✅ | ✅ | ✅ | ⏳ Nem kezdődött |
| 002 | `002-automated-growth-engine` | ✅ | ❌ | ❌ | ❌ |
| 003 | `003-frontend-poc-deployment` | ✅ | ✅ | ✅ | ❌ |
| 004 | `004-infrastructure-and-environments` | ✅ | ❌ | ❌ | ❌ |
| 005 | `005-frontend-dapp` | ✅ | ❌ | ❌ | ❌ |

### Mit kell még csinálni (sorrendben)

1. `002-automated-growth-engine` → `/speckit.plan` → `/speckit.tasks`
2. `004-infrastructure-and-environments` → `/speckit.plan` → `/speckit.tasks`
3. `005-frontend-dapp` → `/speckit.plan` → `/speckit.tasks`
4. Implementáció: `003-frontend-poc-deployment` (PoC app scaffold + Cloudflare deploy)
5. Implementáció: `001` Phase 1–4 (setup → contracts → engine → frontend)
6. Base Sepolia testnet validáció (quickstart.md 6 scenario)
7. Független kód review → mainnet deploy

## Growth Strategy

Initial focus on the Hungarian DeFi community by offering 0% protocol fees for users using the
`HUNGARY` referral code. This serves as our primary "Tracer Bullet" for volume and user feedback.

Ez célzott, ideiglenes promóciós mechanizmus: az alapértelmezett fee továbbra is 0.05%, a 0.00%
csak érvényes referral paraméter mellett használható a kezdeti community bootstrap fázisban.

## Tech Stack

- Frontend: React 19 + Vite + TypeScript
- Wallet integration: wagmi v2 + viem v2
- Backend / routing foundation: Node.js v20+ LTS

---

## A workflow

```text
Constitution → Specify → (Clarify) → Plan → Tasks → Implement
```

| Lépés | Parancs | Mit csinál |
| ----- | ------- | ---------- |
| Constitution | `/speckit.constitution` | Projekt elvek, korlátok, fejlesztési szabályok rögzítése |
| Specify | `/speckit.specify` | Üzleti igény → strukturált feature spec (user stories, stb.) |
| Plan | `/speckit.plan` | Feature spec → technikai terv + kutatás + adatmodellek |
| Tasks | `/speckit.tasks` | Technikai terv → végrehajtható feladatlista |
| Implement | `/speckit.implement` | Feladatlista → implementáció a spec alapján |

## Gyors kezdés VS Code-ban

A fenti parancsok **GitHub Copilot Chat** prompt fájlokként vannak beállítva.
Használatuk Copilot Chatben:

1. Nyisd meg a Copilot Chat panelt (`Ctrl+Alt+I`)
2. Kattints az **Attach context** gombra (papírklip ikon), majd válaszd a **Prompt...**-ot
3. Válaszd ki a kívánt `speckit.*` prompt fájlt
4. Írd be a feature leírását, és küldd el

Vagy közvetlenül hivatkozz a prompt fájlra:

```text
#file:.github/prompts/speckit.specify.prompt.md

Készíts specifikációt erre: [ide jön az üzleti leírás]
```

## Mappastruktúra

```text
.specify/
├── memory/
│   └── constitution.md       ← projekt elvek (először ezt töltsd ki!)
└── templates/
    ├── spec-template.md       ← feature spec sablon
    ├── plan-template.md       ← implementációs terv sablon
    └── tasks-template.md      ← feladatlista sablon

.github/
└── prompts/
    ├── speckit.constitution.prompt.md
    ├── speckit.specify.prompt.md
    ├── speckit.plan.prompt.md
    ├── speckit.tasks.prompt.md
    └── speckit.implement.prompt.md

specs/
└── [###-feature-name]/        ← minden feature külön mappában
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── contracts/
    ├── quickstart.md
    └── tasks.md
```

## Első teendők

> **Ha folytatod a munkát egy másik gépen:**
> 1. `git clone https://github.com/Extracukor/dex-swap-aggregator-spec`
> 2. Nyisd meg VS Code-ban
> 3. Olvasd el a constitution-t és a legutóbbi feature `tasks.md`-jét
> 4. Folytasd a Copilot Chatben a megfelelő prompttal

1. **Töltsd ki a Constitution-t**: nyisd meg [`.specify/memory/constitution.md`](.specify/memory/constitution.md)
   és töltsd ki a placeholdereket a projekt elvekkel.
   Vagy használd a `/speckit.constitution` promptot Copilot Chatben.

2. **Hozd létre az első feature specet**: a `/speckit.specify` promptot használva
   írd le az első üzleti igényt természetes nyelven.

## Spec-kit CLI (később)

Ha Python elérhető lesz, a spec-kit CLI installálható és ráfuttatható erre a repóra:

```powershell
# Python + uv telepítése után:
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# Inicializálás erre a mappára (meglévő fájlokat megtartja):
specify init . --here --force --ai copilot --script ps
```
