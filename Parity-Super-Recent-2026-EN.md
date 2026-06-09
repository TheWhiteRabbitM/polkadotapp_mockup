# Parity / Polkadot — The Super-Recent Repos (2026) & Where Payments Actually Live

> Follow-up focused on the **newest repositories** in **[paritytech](https://github.com/paritytech)**, sorted by **creation date** (via the GitHub API).
> **Analysis date:** 9 June 2026.
> **Correction:** the app with the explicit "Payment in DOT / membership credits" UI I showed earlier (`polkadot-hub-app`) is **old** — its screenshots reference 2023 events. This document covers what is genuinely **new**, and answers honestly where payments/credits live in the new stack.

---

## 1. TL;DR

Parity's 2026 push is a **"super-app" platform**, not a single payments app:

- A **Host** — a Polkadot application (Desktop / Mobile / Browser) — runs sandboxed mini-apps called **products** (webviews).
- Products talk to the Host through the **TrUAPI** protocol and the **product-sdk**.
- Products transact on **Asset Hub** (via `pallet-revive` / Solidity contracts) and store data on the **Polkadot Bulletin chain**; they are addressed by **`.dot` domains** (a Polkadot name service).
- Flagship demo products created this month: **`festival`** (event tickets), **`onchain-arcade`** / **`Rock-Paper-Scissors`** (games), **`t3rminal`** (publish a site to a `.dot` domain), **`simple-survey`**.

> ⚠️ **Honest finding on "the thing that pays and accepts payments":** among the super-recent repos there is **no dedicated payments / commerce / buyer-seller app**. An org-wide search for `pay/payment/checkout/commerce/merchant/credits` returns **0 repositories**. `festival` ticket registration is **free** (soulbound, no `payable` function). The explicit "Pay with DOT (−20%) / membership / time-voucher credits" screens belong to the **older** `polkadot-hub-app` (2023). In the new platform, value moves as **ordinary Asset Hub transactions** mediated by the Host — see §4.

---

## 2. The newest repositories (by creation date)

*Excludes auto-generated `e2e-cli-moddable-*` test fixtures.*

| Repo | Created | Lang | What it is | ELI5 |
|---|---|---|---|---|
| **polkadot-app-design-system-android** | 2026-06-09 | Kotlin | Design tokens for the consumer Polkadot App (Android) | The app's "wardrobe" for Android |
| **festival** ⭐ | 2026-06-08 | TS/Vue/Solidity | Web3 festival app: **soulbound tickets**, attendance **POAPs**, unconference **sessions** | A festival pass + stamps you collect, all on-chain |
| **t3rminal** | 2026-06-08 | TS | Publish a static **Next.js site to a `.dot` domain** (pallet-revive + Bulletin chain) | A website whose address is a blockchain name |
| **t3rminal-OS-test** | 2026-06-06 | TS | Test/variant of t3rminal | Lab version of the above |
| **playground-cli** | 2026-04-15 | TS | Scaffolding/CLI for building products (very actively pushed) | The "create-new-app" tool |
| **onchain-arcade** | 2026-04-08 | TS | **Real-time multiplayer games** on Asset Hub (Tic-Tac-Toe) | Online board games on-chain |
| **Rock-Paper-Scissors** | 2026-04-07 | TS | On-chain game | A game |
| **product-sdk** ⭐ | 2026-03-31 | TS | **SDK to build "products"** for the Host (chain, tx, keys, storage) | The toolkit to build mini-apps |
| **simple-survey** | 2026-03-30 | TS | On-chain survey app | Polls/surveys on-chain |
| **truapi** ⭐ | 2026-03-13 | Rust | **TrUAPI Protocol** docs — host↔product communication (v0.3, 3 Jun 2026) | The "language" host & apps speak |
| **dotli-starter** | 2026-03-06 | HTML | Minimal **LLM-friendly** product starter template | A blank template AI can fill in |
| **host-api-test-sdk** | 2026-03-04 | TS | "**Triangle** e2e Test SDK" (host ↔ product ↔ chain) | Test rig for the platform |
| **subxt-assets** | 2026-02-11 | Rust | CLI for **Paseo Asset Hub** via subxt | Command-line tool for Asset Hub |
| **contract-dependency-manager(-prototype)** | 2026-02/04 | TS | Coordinates dependencies between on-chain contracts | A package manager for smart contracts |

> **ELI5 of the whole thing:** Polkadot is building a phone-like **home screen (the Host)** where you install **mini-apps (products)**. The mini-apps are written with a shared **toolkit (product-sdk)** and talk to the home screen in one shared **language (TrUAPI)**. Each app lives at a blockchain address ending in **`.dot`**.

---

## 3. The platform, piece by piece

### 3.1 The Host
A Polkadot app (e.g. "Polkadot Desktop Browser", plus Mobile) that **hosts products in webviews** and exposes a controlled API surface. `@parity/product-sdk-host` does "Host container detection and storage access for Desktop/Mobile."

### 3.2 Products
Mini-apps (webviews) built with **product-sdk** (TypeScript). The SDK exposes: multi-chain querying, **transaction submission & lifecycle**, **key derivation & account management**, **Asset Hub smart-contract** interactions, **CID-based cloud storage**, and **Statement Store pub/sub**.

### 3.3 TrUAPI — the host↔product protocol
A typed, versioned (wire-ID) RPC protocol. Documented method examples:
- `accountManagement.accountGet({ productAccountId: { dotNsIdentifier: "my-product.dot", derivationIndex: 0 } })`
- `chainInteraction.chainHeadFollow({ genesisHash, withRuntime })`

> 🔑 **Privacy-relevant detail:** accounts are addressed by a **per-product identifier** (`dotNsIdentifier` + `derivationIndex`). Each product effectively gets its **own derived account**, so a product is **scoped/sandboxed** rather than handed your global identity.

### 3.4 The chains
- **Asset Hub** (Paseo testnet) with **`pallet-revive`** running **Solidity** contracts — where product logic & value live.
- **Polkadot Bulletin chain** — stores **metadata / announcements**.
- **`.dot` domains** (dotNs) — human-readable addresses for products/accounts.

### 3.5 Example product: `festival`
- Stack: Nuxt 3 / Vue 3 / TS / Tailwind v4; Solidity 0.8.20 + Foundry + OpenZeppelin v5.5.
- Contracts on Paseo Next V2: `Festival.sol` (inherits **`NonTransferableERC721`** = soulbound tickets + role-based access), plus `FestivalAttendancePOAP` and `SessionAttendancePOAP`.
- Apps: **Admin SPA** (organizer: create festival, check-in, roles), **Attendee SPA** (mobile-first: register, schedule, maps, badges), **Announcements Worker** (posts on-chain announcements).
- **Registration is FREE:** `register()`, `checkIn(address)`, `manualCheckIn(address)` — **none are `payable`**, no `msg.value`, no transfers.

> **ELI5:** A festival app where your ticket is a non-sellable on-chain badge, and you collect "I-was-here" stamps (POAPs) for the festival and each session. Getting a ticket costs nothing.

---

## 4. So… where are the payments / credits / privacy / buyer-seller recharge?

This is the question you care about. Honest, evidence-based answer:

### 4.1 There is **no dedicated payments app** among the super-recent repos
- Org search for `pay/payment/checkout/commerce/merchant/credits` → **0 repos**.
- `festival` tickets are **free** (no payable functions).
- `onchain-arcade` / `Rock-Paper-Scissors` → **no credits, fees, or wagers** (gas optional on testnet).
- `t3rminal` / `simple-survey` → **no payment flows** documented.

### 4.2 Where value *does* move in the new platform
Payments are a **platform capability**, not a separate app:
- Products submit **ordinary Asset Hub transactions** (token transfers, contract calls) via **product-sdk**; signing/keys are mediated by the **Host**.
- Per-product **derived accounts** (TrUAPI) provide **isolation/privacy** between mini-apps.
- Polkadot's published **2026 roadmap** adds **"gas payments in any asset, auto-swapped to DOT under the hood"** — i.e. you can pay fees with stablecoins/other assets and the Host swaps to DOT for you.

> **ELI5:** The new apps don't each build their own till. When an app needs to move money, it asks the Host to make a normal Polkadot payment, and the Host can let you pay fees in whatever coin you hold (it swaps to DOT behind the scenes). Each app only sees its own sandboxed account, not your whole wallet.

### 4.3 The explicit "buyer/seller, credits, recharge" UI you remembered = the **older** Hub App
The screens that literally showed **Pay with DOT (−20%)**, **soulbound membership**, **time-voucher (day/month/year) credits**, **connect wallet → mint voucher** are from **`polkadot-hub-app`** — a **2023** internal "Parity HQ" app (its screenshots reference 2023 events). That model:
- **Credits:** DOT (payment) + free **soulbound membership NFT** (identity) + **time-voucher NFTs** (spendable access).
- **Customer recharge:** connect own wallet → pay DOT (fiat→DOT at sale) → receive voucher NFT.
- **Seller settlement:** the hub's contract/treasury receives DOT and mints the voucher.
- **Privacy:** non-custodial wallets, login-with-Polkadot, self-hosted, stealth mode.

*(Full screenshot tour of that older app is in **`Parity-Polkadot-Deep-Analysis-EN.md`** + the deck — just be aware it is the 2023 app, not the new platform.)*

### 4.4 Net answer
If you were hoping the **buyer/seller + credits + recharge** app is brand-new: **it isn't published as such yet.** The 2026 direction is a **super-app Host** where any product can transact on **Asset Hub** through a shared SDK, with **per-product account isolation** for privacy and **pay-fees-in-any-asset** on the roadmap. A consumer-facing **commerce** experience (with cashback/credits) is the separately announced **consumer Polkadot App + Raise** — but **that code is not in these repos** (only design tokens are).

---

## 5. Screenshots? Not in these repos
Unlike the 2023 `polkadot-hub-app` (which ships 14 PNGs in `docs/images`), the **super-recent repos contain no screenshots** in their READMEs (`festival`, `t3rminal`, `onchain-arcade`, `product-sdk`, `truapi` — checked, none). Any visual deck for the **new** platform would therefore be **diagram/text-based**, not real app captures. (The real captures we *do* have remain those of the older Hub App.)

---

## 6. Sources
- [paritytech org — repos by creation date (API)](https://api.github.com/orgs/paritytech/repos?sort=created&direction=desc&per_page=30)
- [paritytech/festival](https://github.com/paritytech/festival) · [Festival.sol](https://github.com/paritytech/festival/blob/main/contracts/src/apps/events/Festival.sol)
- [paritytech/t3rminal](https://github.com/paritytech/t3rminal) · [paritytech/onchain-arcade](https://github.com/paritytech/onchain-arcade)
- [paritytech/product-sdk](https://github.com/paritytech/product-sdk) · [paritytech/truapi](https://github.com/paritytech/truapi) (TrUAPI v0.3) · [docs site](https://paritytech.github.io/truapi/)
- [Polkadot Roundup 2025 — Parity blog](https://www.parity.io/blog/polkadot-roundup-2025) (2026 = "products + a platform for them to sit upon"; Hub roadmap: gas in any asset auto-swapped to DOT)
- [What is Polkadot Hub? — Parity blog](https://www.parity.io/blog/what-is-polkadot-hub)

---

*Independent analysis from public sources; contract behavior verified by reading the Solidity source. Where official sources are silent, this is stated explicitly. Not financial advice.*
