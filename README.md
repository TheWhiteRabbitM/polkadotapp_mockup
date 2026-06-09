# 🐇 Polkadot Super-App — Concept Mock-ups (`TWR.DOT`)

> A black-and-white, cyberpunk reconstruction of **Polkadot's emerging "self-sovereign super-app"**, imagined by **reading the open-source repositories** of the [paritytech](https://github.com/paritytech) GitHub organization.
> Wallet/identity used throughout: **`TWR.DOT`**. · *Concept only — see [Disclaimer](#-disclaimer).*

**Live page:** `index.html` (single self-contained file, ready for **GitHub Pages**).

---

## What this is

Parity's newest open-source repos (Feb–Jun 2026) don't ship a finished consumer app — they ship the **protocol, the SDK, and a handful of reference "products."** There are **no real screenshots** of the super-app anywhere in the repos.

So we did the next best thing: we **read the protocol and the contracts**, inferred what the app *must* look and feel like, and **drew it** — ten concept phone screens, each annotated with the exact protocol primitive it would call. The result is a slick, presentation-style page in a strict **black & white cyberpunk** aesthetic (CRT scanlines, grid, glitch, terminal cues — and a white rabbit).

---

## Repo contents

| File | What it is |
|---|---|
| **`index.html`** | The GitHub Pages site: hero + 10 concept screens + white rabbit + disclaimer |
| `Polkadot-SuperApp-Deck.html` | 16-slide analyst presentation (B/W cyberpunk) |
| `Polkadot-SuperApp-Mockups.html` | The raw mock-up gallery (pre-presentation version) |
| `Parity-Analyst-FieldNotes-2026.md` | The deep analyst write-up the concept is based on |
| `Parity-Super-Recent-2026-EN.md` | Inventory & analysis of the newest repos |
| `Parity-Polkadot-Deep-Analysis-EN.md` | Earlier deep-dive (incl. the older 2023 `polkadot-hub-app`) |

---

## The thesis (in one sentence)

> A native **Host** (the Polkadot App: Desktop / Mobile / Web) runs sandboxed mini-apps called **products**, which talk to the device and the chain through an OS-grade API called **TrUAPI** — *"Triangle User-Agent Programming Interface."*
> Think **WeChat Mini-Programs × Telegram Mini Apps × iOS capability-permissions × a self-custodial wallet**, on-chain, addressed by **`.dot`** names.

```
        ┌───────────────┐
        │     HOST       │   keys · identity (.dot) · UI · consent
        └──────▲─────────┘
               │  TrUAPI  (SCALE over MessagePort)
        ┌──────▼─────────┐
        │   PRODUCTS      │   sandboxed mini-apps · @parity/product-sdk
        └──────▼─────────┘
        ┌────────────────┐
        │   CHAINS        │   Asset Hub · Bulletin · Statement Store
        └────────────────┘
```

The "Triangle" = **Host ↔ Product ↔ Chain** (the same triangle named in `host-api-test-sdk`, the "Triangle e2e Test SDK").

---

## How we imagined the app (methodology)

We treated the **TrUAPI method surface** as the source of truth: if the protocol exposes a capability, the app must have a UI for it. Each screen below is a **direct read** of a real primitive found in the repos.

| # | Screen | Imagined because the protocol exposes… |
|---|---|---|
| 01 | **Onboarding** (`.dot` / Google / import) | `host_request_login`, `host_get_user_id` (primary DotNS identity) |
| 02 | **Home / Wallet** — balance = "credits" | `host_payment_balance_subscribe` (a host-managed balance) |
| 03 | **Top-up** (recharge buyer, auto-swap → DOT) | `host_payment_top_up` + roadmap "pay fees in any asset → DOT" |
| 04 | **Pay the seller** (tap-to-pay, Face ID) | `host_payment_request` → `PaymentId`; `DevicePermission::Nfc`, `Biometrics` |
| 05 | **JIT permission** ("Allow always / once / never") | `host_device_permission` (iOS-style consent) |
| 06 | **Mini-app store** ("products") | `@parity/product-sdk-*`, products as sandboxed webviews |
| 07 | **festival** — soulbound ticket + POAPs | `Festival.sol : NonTransferableERC721`; Bulletin storage |
| 08 | **arcade** — real-time multiplayer | `remote_statement_store_*` (pub/sub presence) |
| 09 | **Chat (E2E)** | `host_chat_create_simple_group` + `host_derive_entropy` → X25519 |
| 10 | **t3rminal** — publish a site to `.dot` | Bulletin Chain (CIDv1 Blake2b-256) + `pallet-revive` on-chain index |

### Why `TWR.DOT` matters in the design
Identity is **scoped**: you sign in once as `TWR.DOT`, but **every product gets its own derived account** — it never sees your global wallet:

```ts
accountGet({ productAccountId: { dotNsIdentifier: "my-app.dot", derivationIndex: 0 } })
```

That single design choice (per-product derived accounts + just-in-time permissions + on-device `derive_entropy` keys) is what makes the concept **privacy-by-architecture**, not privacy-by-promise.

---

## The full TrUAPI capability surface (what we found)

| Namespace | Methods (verbatim) |
|---|---|
| **Identity** | `host_get_user_id`, `host_request_login`, `host_account_get`, `host_get_legacy_accounts` |
| **Signing / Tx** | `host_sign_payload`, `host_sign_raw`, `host_create_transaction_*` (+ legacy variants) |
| **Device perms (JIT)** | `host_device_permission` → `Notifications`, `Nfc`, `Clipboard`, `OpenUrl`, `Biometrics`, … |
| **Remote perms** | `remote_permission` → `Remote(domains)`, `WebRtc`, `ChainSubmit`, `StatementSubmit` |
| **Payments** | `host_payment_balance_subscribe`, `host_payment_top_up`, `host_payment_request`, `host_payment_status_subscribe` |
| **Crypto** | `host_derive_entropy` (BLAKE2b → X25519, scoped to product) |
| **Chat** | `host_chat_create_simple_group` |
| **Statement Store** | `remote_statement_store_subscribe` (`MatchAll`/`MatchAny`), `remote_statement_store_submit` |
| **Theme** | `host_theme_subscribe` (`Light`/`Dark`) |

**`product-sdk`** wraps this with 15 packages (`@parity/product-sdk-*`): `chain-client`, `tx`, `signer`, `contracts` (typed pallet-revive/Solidity), `cloud-storage` (Bulletin/CID), `statement-store`, `keys` (sr25519 product accounts), `address` (SS58 ↔ **H160**, i.e. EVM-friendly), `crypto` (NaCl), `host`, and more. It even ships a **`.claude-plugin/` + `CLAUDE.md`** — a strong signal the catalog is meant to be **agent-built**.

---

## Sources

All conclusions are drawn from public, open-source material; protocol/method names and contract behaviour were read directly from source.

**Core platform**
- [paritytech/truapi](https://github.com/paritytech/truapi) — TrUAPI protocol · [v0.2 design notes](https://github.com/paritytech/truapi/blob/main/docs/design/releases/v0.2.md) · [Rust docs](https://paritytech.github.io/truapi/) · playground `truapi-playground.dot.li`
- [paritytech/product-sdk](https://github.com/paritytech/product-sdk) — the product TypeScript SDK (15 packages)
- [paritytech/host-api-test-sdk](https://github.com/paritytech/host-api-test-sdk) — "Triangle e2e Test SDK"

**Reference products**
- [paritytech/festival](https://github.com/paritytech/festival) — events / soulbound tickets / POAPs · [`Festival.sol`](https://github.com/paritytech/festival/blob/main/contracts/src/apps/events/Festival.sol)
- [paritytech/onchain-arcade](https://github.com/paritytech/onchain-arcade) · [paritytech/Rock-Paper-Scissors](https://github.com/paritytech/Rock-Paper-Scissors)
- [paritytech/simple-survey](https://github.com/paritytech/simple-survey)
- [paritytech/t3rminal](https://github.com/paritytech/t3rminal) — publish a static site to a `.dot` domain

**Dev experience / tooling**
- [paritytech/dotli-starter](https://github.com/paritytech/dotli-starter) — "LLM-friendly" product starter
- [paritytech/playground-cli](https://github.com/paritytech/playground-cli) — scaffolding CLI
- [paritytech/subxt-assets](https://github.com/paritytech/subxt-assets) · [paritytech/contract-dependency-manager](https://github.com/paritytech/contract-dependency-manager)

**Host UI**
- [paritytech/polkadot-app-design-system](https://github.com/paritytech/polkadot-app-design-system) (+ `-android`, `-ios`, `-desktop`)

**Context / strategy**
- [Polkadot Roundup 2025 — Parity blog](https://www.parity.io/blog/polkadot-roundup-2025) (2026 = "products + a platform for them to sit upon"; gas in any asset auto-swapped to DOT)
- [What is Polkadot Hub? — Parity blog](https://www.parity.io/blog/what-is-polkadot-hub)
- Repos by creation date (API): `https://api.github.com/orgs/paritytech/repos?sort=created&direction=desc`

---

## Deploy to GitHub Pages

1. Create a repo and add **`index.html`** (already named for the site root).
2. **Settings → Pages → Build and deployment → Source: `Deploy from a branch` → `main` / `/root`** → **Save**.
3. The site goes live at `https://<user>.github.io/<repo>/`.

No build step, no dependencies — system fonts only, so it looks identical everywhere.

---

## ⚠ Disclaimer

These are **illustrative concept mock-ups**, hand-built from publicly available **open-source** material in the **paritytech** GitHub organization. They are **not real screenshots**, **not official Parity / Polkadot / Web3 Foundation designs**, and are **not affiliated with or endorsed by** them. All names, balances, merchants and the **`TWR.DOT`** identity are **fictional** and for demonstration only. The underlying repositories are **experimental, testnet-only and explicitly unaudited**; methods and behaviour may change at any time. Trademarks belong to their respective owners. **Nothing here is financial, investment, legal or security advice.**

---

*🐇 follow the white rabbit.*
