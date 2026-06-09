# Field Notes of a Web3 Analyst — What Parity Is *Actually* Building (June 2026)

> A close reading of the **newest open-source repositories** in **[github.com/paritytech](https://github.com/paritytech)**, written as if briefing a sharp colleague (and, in spirit, a friend of Gavin's). I read the **protocol definitions and contract source**, not just the READMEs. Confirmed facts are marked plainly; my inferences are labelled **HYPOTHESIS**.
> **Date:** 9 June 2026 · **Scope:** repos created Feb–Jun 2026, public/OSS only.

---

## 0. The one-line thesis

> **Polkadot is building a self-sovereign "super-app".** A native **Host** (the Polkadot App / "Desktop Browser") runs sandboxed **mini-apps called "products"**, which talk to the device and the chain through an **OS-grade API called TrUAPI**. Think **WeChat Mini-Programs × Telegram Mini Apps × iOS capability-permissions × a built-in crypto wallet** — but self-custodial, on-chain, and addressable by **`.dot` names**.

Everything new in the org over the last quarter is a piece of *this*: the Host, the SDK to build products, the protocol between them, the storage/identity layer underneath, and a handful of **reference products** to prove it works.

---

## 1. The evidence base (what's new, by creation date)

| Repo | Created | Role in the stack | Layer |
|---|---|---|---|
| **truapi** | 2026-03-13 (v0.3, 3 Jun) | The **host↔product protocol** ("Triangle User-Agent Programming Interface") | OS API |
| **product-sdk** | 2026-03-31 | The **TypeScript toolkit** to build products | SDK |
| **host-api-test-sdk** | 2026-03-04 | "**Triangle** e2e Test SDK" — host ↔ product ↔ chain | Test harness |
| **playground-cli** | 2026-04-15 | Scaffolding/CLI (very actively pushed) | DX |
| **dotli-starter** | 2026-03-06 | **LLM-friendly** product starter template | DX (AI-native) |
| **festival** | 2026-06-08 | Reference product: events (soulbound tickets, POAPs, sessions) | Product |
| **onchain-arcade** / **Rock-Paper-Scissors** | 2026-04 | Reference products: real-time multiplayer games | Product |
| **simple-survey** | 2026-03-30 | Reference product: on-chain surveys | Product |
| **t3rminal** / **t3rminal-OS-test** | 2026-06-08 | Publish a static site to a **`.dot` domain** (decentralized web) | Product / infra |
| **polkadot-app-design-system** (+ android/ios/desktop) | 2026-06-09 | The **Host's UI** design tokens | Host UI |
| **subxt-assets**, **contract-dependency-manager** | 2026-02 | Asset-Hub CLI; on-chain dependency coordination | Infra |

The auto-generated `e2e-cli-moddable-*` repos are CI fixtures from `playground-cli` — ignore.

---

## 2. The architecture, decoded

```
        ┌──────────────────────────────────────────────────────┐
        │                      THE HOST                          │
        │      (Polkadot App: Desktop / Mobile / Browser)        │
        │  • holds the user's keys & identity (DotNS .dot)       │
        │  • mediates capabilities via JIT permission prompts    │
        │  • renders chrome: chat UI, theme, payment sheets      │
        └───────────────▲───────────────────────┬───────────────┘
                        │  TrUAPI (SCALE over    │
                        │  MessagePort/postMessage)
        ┌───────────────┴───────────────────────▼───────────────┐
        │                     PRODUCTS                            │
        │     sandboxed webviews = "mini-apps"                    │
        │  festival · arcade · survey · t3rminal · (your app)    │
        │     built with @parity/product-sdk-*                   │
        └───────────────┬───────────────────────────────────────┘
                        │ chain ops (PAPI / pallet-revive)
        ┌───────────────▼───────────────────────────────────────┐
        │   CHAINS:  Asset Hub (Solidity via pallet-revive)      │
        │            Bulletin Chain (storage, CIDv1 Blake2b-256) │
        │            Statement Store (pub/sub messaging)         │
        └────────────────────────────────────────────────────────┘
```

The **"Triangle"** in TrUAPI (and in the *Triangle e2e Test SDK*) is exactly these three corners: **Host ↔ Product ↔ Chain**. That is the mental model the whole codebase is built around.

---

## 3. TrUAPI — the operating-system API (the heart of the story)

TrUAPI is "the API surface that hosts like the **Polkadot Desktop Browser** expose to the **products** that run inside them." One Rust crate defines the contract; a generator emits a typed TypeScript client; **hosts and products implement the same shared types.** Dispatch is by **append-only wire IDs** (`#[wire(id = N)]`) so deployed products never break across protocol revisions. Transport is **SCALE frames over `MessagePort`** (or iframe `postMessage`).

This is not a wallet-connect shim. The v0.2 surface reads like a **mobile OS SDK**. The namespaces:

| Namespace | Methods (verbatim) | What it unlocks |
|---|---|---|
| **Accounts / Identity** | `host_get_user_id` (primary **DotNS** identity, JIT-approved), `host_request_login`, `host_account_get` (**per-product** derived account), `host_get_legacy_accounts` | Single sign-on + **per-app account isolation** |
| **Signing / Tx** | `host_sign_payload`, `host_sign_raw`, `host_create_transaction_*`, legacy-account variants | Products never touch keys; the Host signs after consent |
| **Device permissions** (JIT) | `host_device_permission` → `Notifications`, **`Nfc` (tap-to-pay)**, `Clipboard`, `OpenUrl`, **`Biometrics`** (+ more) | Native phone capabilities, iOS-style "Allow once / always / never" |
| **Remote permissions** | `remote_permission` → `Remote(domain patterns)`, `WebRtc`, `ChainSubmit`, `StatementSubmit`, `PreimageSubmit` | Gates network, chain broadcast, P2P |
| **💳 Payments** | `host_payment_balance_subscribe`, **`host_payment_top_up`**, **`host_payment_request`** (→ `PaymentId`), `host_payment_status_subscribe` (`Processing→Completed/Failed`) | A **wallet-balance "credits" layer** with top-up + request-to-pay |
| **🔐 Entropy / Crypto** | `host_derive_entropy` (32 bytes, scoped to product+key, 3-layer BLAKE2b → **X25519** keys) | **End-to-end encryption / P2P** without round-trips |
| **Chat** | `host_chat_create_simple_group` (returns a join link; Host renders UI) | Native social/messaging primitive |
| **Statement Store** | `remote_statement_store_subscribe` (topic `MatchAll`/`MatchAny`), `remote_statement_store_submit` | Decentralized **pub/sub** event bus |
| **Theme** | `host_theme_subscribe` (`Light`/`Dark`) | Products inherit Host look-and-feel |

> **Read this table again.** Identity + signing + NFC + biometrics + **a payment balance you can top-up and request payments against** + E2E encryption + chat + pub/sub. That is the capability set of a **consumer financial-social super-app**, not a developer tool.

### 3.1 This is the answer to the "payments / credits / privacy / recharge" question

The thing you sensed earlier — *something that pays and accepts payments, with credits, privacy, and recharging buyer & seller accounts* — **is real, and it lives here, at the Host/protocol level**, not in any single app:

- **"Credits" / balance** → `host_payment_balance_subscribe`: the user has a **Host-managed balance**. (Matches Polkadot's 2026 roadmap line: *pay fees in any asset, auto-swapped to DOT under the hood* — the Host abstracts the token.)
- **Recharge the customer's account** → **`host_payment_top_up`** ("top-ups user balance from a product-controlled source").
- **Pay the seller** → **`host_payment_request`** ("requests a payment from the user **to a destination**; prompts for authorization; returns a `PaymentId`"). The destination is the merchant/product = the **seller's account**.
- **Privacy** → **per-product derived accounts** (`productAccountId = { dotNsIdentifier, derivationIndex }`) so a merchant app sees a **scoped** account, not your global identity; **JIT capability permissions**; **`host_derive_entropy` → X25519** for **E2E-encrypted** product-to-product/peer data.

So the earlier `polkadot-hub-app` (2023) was just an *early, hard-coded* version of this idea (pay DOT, hold membership/voucher NFTs). The **2026 platform generalises it into an OS primitive** any mini-app can call. That is the real upgrade.

---

## 4. product-sdk — the toolkit that makes products cheap to build

A TypeScript monorepo of **15 packages** (`@parity/product-sdk-*`). The Host does the hard parts (keys, consent, payments); the SDK gives products clean handles:

| Package | Purpose |
|---|---|
| `…-chain-client` | Multi-chain PAPI access (Asset Hub, Bulletin, …) |
| `…-tx` | Transaction submission & lifecycle |
| `…-signer` | Multi-provider signer (Host API + dev providers) |
| `…-contracts` | Typed **Asset Hub / pallet-revive (Solidity)** calls |
| `…-cloud-storage` | **CID-based** upload/retrieve, backed by **Bulletin Chain** |
| `…-statement-store` | Pub/sub client |
| `…-keys` | Hierarchical derivation, **session keys**, sr25519 **product-account** derivation |
| `…-local-storage` | KV store, auto host/browser backend |
| `…-host` | Host container detection (Desktop/Mobile) |
| `…-address` | SS58 / H160 encode-validate-convert |
| `…-crypto` | Encryption, key derivation, **NaCl** |
| `…-descriptors` | PAPI-generated chain descriptors |
| `…-logger`, `…-utils`, `…-product-sdk` (umbrella) | DX glue |

**Signal:** `SS58 / H160` conversion + `pallet-revive` + typed contracts means the platform is **deliberately EVM-Solidity-friendly** (H160 = Ethereum-style addresses), lowering the barrier for the millions of Solidity devs. This dovetails with **`revive`** (Solidity→PolkaVM) and **Polkadot Hub** (EVM + RISC-V).

---

## 5. The data, identity & web layer

- **DotNS / `.dot` domains** — human-readable identity & addressing for users *and* products (`my-product.dot`). This is Polkadot's **name service as the primary key** of the whole UX. The playground itself lives at `truapi-playground.dot.li`; **`dot.li`** looks like the **gateway** that resolves `.dot` to the web.
- **Bulletin Chain** (`pallet-transaction-storage`, **CIDv1 Blake2b-256**) — cheap, content-addressed **decentralized storage**. Used by `cloud-storage`, by `festival` for announcements/metadata, and by `t3rminal` to host whole websites.
- **Statement Store** — a **gossip/pub-sub** layer for off-chain-ish messages with topic filters; the backbone for **chat, presence, and real-time** product features (the games, festival announcements).
- **t3rminal** — the boldest one: **publish a static Next.js site to a `.dot` domain**, stored on Bulletin, indexed by a `pallet-revive` contract (`T3rminalBulletinIndex`). **HYPOTHESIS:** this is Polkadot's play for a **fully decentralized web** (IPFS-style hosting + ENS-style naming + on-chain index), where "a website" is an on-chain object you own.

---

## 6. The AI-native developer experience (don't miss this)

Three quiet but loud signals:

1. **`product-sdk` ships a `.claude-plugin/` and a `CLAUDE.md`.**
2. **`dotli-starter`** is explicitly a *"Minimal **LLM-friendly** starter prototype template."*
3. **`playground-cli`** auto-scaffolds and even auto-generates throwaway E2E fixtures.

**HYPOTHESIS:** Parity is optimising for **AI agents building mini-apps**. The bet: if a coding agent can scaffold a compliant "product" against a typed SDK + a stable protocol in minutes, the **mini-app catalog fills itself**. This is the same flywheel WeChat/Telegram got from low-friction mini-program tooling — accelerated by LLMs. It also explains the "LLM-friendly" framing: typed, versioned, self-describing APIs are exactly what agents need.

---

## 7. The reference products — and what each one is really proving

| Product | On the tin | What it's actually a proof-of | 
|---|---|---|
| **festival** | Event tickets + POAPs + sessions | **Soulbound identity & attestations** + Bulletin storage + chat announcements → the "**events/loyalty**" vertical. Tickets are *free* (no `payable`) — the point is **identity, not commerce** here. |
| **onchain-arcade / RPS** | Multiplayer games | **Real-time state via Statement Store** + per-product accounts → the "**social/gaming**" vertical (and latency proof). |
| **simple-survey** | Surveys | **Cheap writes + identity** → "**data collection / governance / Sybil-resistant polls**". |
| **t3rminal** | Publish to `.dot` | **Decentralized web hosting + naming** → the "**own your site/app**" vertical. |
| **polkadot-app-design-system** | Tokens for iOS/Android/desktop | The **Host's** consistent native UI across platforms. |

Taken together they tile the surface of a consumer super-app: **identity, payments, social, gaming, content, commerce** — each "vertical" demoed by one small, deliberately-scoped product.

---

## 8. Strategic read — the Gavin Wood lens

Gavin's long arc has been: **Ethereum (general compute) → Polkadot (shared security, parachains) → Polkadot 2.0 (agile coretime, elastic scaling) → JAM (the Join-Accumulate world computer).** That's all **infrastructure**. The conspicuous gap was always the **application layer and the user** — crypto's eternal UX problem.

These repos are the **application-layer answer**, and they're philosophically on-brand:

- **"Abstract the chain away."** Users get a `.dot` name, a balance, biometric approval, and tap-to-pay. They never see gas, RPC, or seed phrases. (The roadmap's *pay-fees-in-any-asset-auto-swap-to-DOT* is the economic half of this.)
- **Self-sovereign, not custodial.** Keys stay with the Host/user; products are sandboxed and get **derived, scoped accounts** + **JIT capability grants**. It's the **capability-security model** (think object-capabilities — very Gavin) applied to apps.
- **"Tru/Triangle" = minimised trust.** The Host mediates; products can't exceed granted capabilities; the protocol is versioned and append-only so trust assumptions don't silently drift.
- **On-chain everything that can be.** Identity (DotNS), storage (Bulletin), messaging (Statement Store), logic (pallet-revive) — the web rebuilt on Polkadot primitives.

**HYPOTHESIS (the big one):** This is Polkadot's **"iPhone moment" attempt** — not another L1 narrative, but a **consumer gateway** that turns Polkadot's infra into something a non-crypto person taps to pay for coffee, buys an event ticket, plays a game, and runs a website — all from one self-custodial app, with a mini-app store that **AI agents can stock**. If it lands, the moat isn't TPS; it's **the account, the `.dot` identity, and the payment balance** sitting in millions of pockets.

---

## 9. What to watch / open questions (analyst's caveats)

- **Custody & off-ramp of the payment balance.** `host_payment_top_up` "from a product-controlled source" is powerful but under-specified — is the balance DOT, a stablecoin, or Host-IOU credits? Who can top-up programmatically, and what stops abuse?
- **KYC/AML & fiat rails.** NFC tap-to-pay + payment requests at consumer scale eventually meets regulation. No sign of it in the OSS yet.
- **Where the Host itself lives.** I did **not** find a public `polkadot-host` / "Desktop Browser" repo — the Host may be closed or in another org. The OSS is the **protocol + SDK + products**, i.e. the *open* side of a possibly-curated app platform. **Watch for the Host repo / app-store policy** — that's where "permissionless" gets tested.
- **Everything is `Paseo` testnet + "not audited."** This is **pre-product** (prototype/PoC disclaimers everywhere). Timelines and final shape can still move.
- **Statement Store at scale** (real-time games, presence) is an ambitious use of a gossip layer; latency/abuse are open.

---

## 10. Bottom line

The newest open-source repos are **not** a grab-bag of demos — they are the **coherent first release of an application platform**:

> **Host + Products + TrUAPI + product-sdk + DotNS + Bulletin + Statement Store = a self-custodial Polkadot super-app with a mini-app economy.** The payments you were chasing are a **first-class OS primitive** (`host_payment_top_up` / `host_payment_request` / balance), privacy comes from **per-product scoped accounts + JIT permissions + on-device key derivation**, and the developer flywheel is **explicitly AI-assisted**. If Parity ships the Host to mainstream users, this is the most consequential thing they've built since parachains.

---

## 11. Sources (primary; code/protocol read directly)
- [paritytech/truapi](https://github.com/paritytech/truapi) · [v0.2 design notes](https://github.com/paritytech/truapi/blob/main/docs/design/releases/v0.2.md) · [Rust docs](https://paritytech.github.io/truapi/) · playground: `truapi-playground.dot.li`
- [paritytech/product-sdk](https://github.com/paritytech/product-sdk) (15 `@parity/product-sdk-*` packages; ships `.claude-plugin/` + `CLAUDE.md`)
- [paritytech/festival](https://github.com/paritytech/festival) · [Festival.sol](https://github.com/paritytech/festival/blob/main/contracts/src/apps/events/Festival.sol) (registration is **free**, soulbound) · DEPLOY.md
- [paritytech/t3rminal](https://github.com/paritytech/t3rminal) (publish to `.dot`; `T3rminalBulletinIndex`, Bulletin CIDv1 Blake2b-256)
- [paritytech/onchain-arcade](https://github.com/paritytech/onchain-arcade) · [Rock-Paper-Scissors](https://github.com/paritytech/Rock-Paper-Scissors) · [simple-survey](https://github.com/paritytech/simple-survey)
- [paritytech/dotli-starter](https://github.com/paritytech/dotli-starter) ("LLM-friendly") · [playground-cli](https://github.com/paritytech/playground-cli) · [host-api-test-sdk](https://github.com/paritytech/host-api-test-sdk) ("Triangle e2e Test SDK")
- [paritytech/subxt-assets](https://github.com/paritytech/subxt-assets) · [contract-dependency-manager](https://github.com/paritytech/contract-dependency-manager)
- Repos by creation date (API): `https://api.github.com/orgs/paritytech/repos?sort=created&direction=desc`
- Context: [Polkadot Roundup 2025 — Parity blog](https://www.parity.io/blog/polkadot-roundup-2025) (2026 = "products + a platform for them to sit upon"; gas-in-any-asset → DOT)

---

*Independent analysis. Protocol/method names and contract behaviour were read directly from source; strategic claims are labelled **HYPOTHESIS**. Everything cited is pre-production, testnet, and explicitly unaudited. Not investment advice.*
