# 🚨 The Host Is Open — `polkadot-ios-community` Analysis

> **Date:** 10 June 2026 · Repo: [paritytech/polkadot-ios-community](https://github.com/paritytech/polkadot-ios-community)
> **What happened:** the piece our analysis flagged as *"named on the map but 404 — watch the release queue"* just landed. This is the **iOS Host** — the actual **Polkadot self-custodial super-app** — published as a GPL-3.0 community/reference release.
> Swift 97% · iOS 17+ · 28 local Swift packages · VIPER · substrate-sdk-ios · targets **Paseo** testnet · explicitly **prototype, unaudited**.

---

## 1. Why this matters

Our whole thesis ("Host + Products + TrUAPI = self-sovereign super-app") was reconstructed from SDKs, protocol docs and the `w3s-architecture` map. **This repo is the thesis, shipped.** The README literally describes a self-custodial super-app with identity, chat, calls, payments, and sandboxed dApps. The release queue is moving: `identity-backend-community` → **`polkadot-ios-community`** → (next: `polkadot-app-android-v2`? `polkadot-desktop`? `w3spay`?).

## 2. What's inside (from the README, verbatim themes)

### Identity — three rungs of personhood
1. **On-chain username** registered via the **People Chain**
2. **Proof-of-Unique-Device** → grants **transaction allowances** *(a device attestation that buys you fee headroom — onboarding without "first, buy DOT")*
3. **Proof-of-Personhood elevation** via **"DIM2 videocall gesture gaming"** — PoP by playing gesture games on a video call. This is the `individuality` repo (and dotNS's PopLite/PopFull tiers) made flesh.

### Communications — the chat half of the super-app
- **E2E-encrypted P2P messaging** with media
- **Encrypted voice & video calls over WebRTC**, peer-to-peer (recall TrUAPI's `remote_permission::WebRtc`)
- Messages ride the **People Chain statement infrastructure** (Statement Store as a messaging bus!)

### Payments — answers to the original four questions, now in the Host
- **Username-based payment routing** (usernames resolve to on-chain accounts — pay `alice`, not `5Grw…`)
- **QR payments** + **in-chat transfers**
- **Auto-conversion via Asset Hub liquidity pools** (pay with what you hold; pools swap it) — and there's a **`HydrationSdk`** package, so the Hydration DEX is in the conversion path too
- **Fiat on-ramp** hooks

### dApps ("products") — the store half
- **Sandboxed dApp execution with granular permissions**
- Two modalities: **SPA** *and* **Chat** — dApps that live *inside conversations* (Telegram-bot-style, but sandboxed and on-chain)
- **DotNS addressing**, content hosted on **Bulletin Chain**, deep-linking via URL/QR
- A literal **`Products`** package

### Cross-device — the phone is the key
- **Remote signing** for **Polkadot Desktop and Web**: pairing via QR; your phone stays the **signing authority** (WhatsApp-Web pattern, applied to keys)
- Contact/chat sync across devices (**`HandoffService`**)

### Custody & backup
- **iCloud Keychain** or **Google Drive** *encrypted* backups, or **secure-enclave-only**
- The line that defines the product: *“There is no custodian, and nobody… can freeze, recover, or move your account.”*

## 3. The 28 packages — the architecture in nouns

`AssetExchange · AssetHubSdk · AssetsManagement · BulletinChain · CarParser · ChainStore · Coinage · HandoffService · HydrationSdk · **Individuality** · JailbreakDetection · KeyDerivation · MessageExchangeKit · PolkadotUI · **Products** · StatementStore · SubstrateOperation · XcmDefinition · XcmTransfer · …`

Reading the names: **Individuality** (PoP), **Products** (mini-apps), **StatementStore/MessageExchangeKit** (chat rails), **AssetExchange/HydrationSdk/XcmTransfer** (payments & cross-chain), **BulletinChain + CarParser** (CAR = IPFS Content-ARchive parsing → content-addressed dApp hosting), **HandoffService** (desktop pairing), **JailbreakDetection** (host integrity), **PolkadotUI** (the design system we saw as token repos).

## 4. Updates to the original questions

| Question | Now confirmed by the Host |
|---|---|
| **Credits** | On-chain balances + **allowances earned by Proof-of-Unique-Device**; any asset spendable via **pool auto-conversion** |
| **Privacy** | Self-custody (enclave), **E2E chat & WebRTC calls**, sandboxed dApps with granular permissions, device attestation instead of accounts |
| **Buyer recharge** | **Fiat on-ramp** + QR/username receive + (testnet) faucet |
| **Seller settlement** | **Username → on-chain account** routing; in-chat payment requests; conversion handled by pools |

## 5. Caveats (theirs and ours)
- **Prototype / PoC / unaudited**, may contain vulnerabilities — their words.
- Targets **Paseo**; chain config arrives via **remote configuration** (a centralization knob to watch).
- Optional **Firebase/analytics/crash** hooks exist (default-off "safe public configuration") — worth auditing in a privacy app.
- TrUAPI isn't named in the README; the dApp sandbox/permissions clearly play that role — naming may differ in code.
- One squashed commit on `main` — a curated community drop, not the live internal history.

## 6. Bottom line
The 404s were the release queue, and the queue is shipping. With the **iOS Host now public**, the open stack reads: **Host (iOS) + product-sdk + TrUAPI + dotNS + Bulletin + People + identity-backend** — everything our deck claimed, now verifiable in Swift. Next watch-items: **Android Host, Desktop, and the W3S Pay trio.**

*Unaudited prototype · testnet · not financial advice · unaffiliated with Parity.*
