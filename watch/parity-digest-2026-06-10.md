# Parity / paritytech Org Watch — Digest

**Date of scan:** 2026-06-10
**Baseline:** No prior digest found — this is the **first run**. Treating ~3-day window (since 2026-06-07) as "new"; older items within 7 days listed and marked.

## New repositories (created in the last ~3 days)

| Name | Created | Lang | One-line description |
|------|---------|------|----------------------|
| **identity-backend-community** 🆕 | 2026-06-10 | TypeScript | Backend-for-frontend for the Polkadot mobile app; reference identity/auth services (experimental, unaudited) |
| polkadot-app-deploy | 2026-06-09 | JavaScript | (no description) — deploy tooling for the Polkadot app |
| playground-app-community | 2026-06-09 | TypeScript | (no description) — community playground app |
| w3s-architecture | 2026-06-09 | HTML | Open-sourcing map + reference repo list for inspecting "W3S" (Web3 Services) project sources |
| polkadot-app-design-system-android | 2026-06-09 | Kotlin | (no description) — Android port of the Polkadot app design system |
| festival | 2026-06-08 | TypeScript | Web3 festival mgmt prototype on Polkart: soulbound tickets, proof-of-attendance, runs in Polkadot Host |
| w3s-payment-processor | 2026-06-08 | TypeScript | Merchant Web3 payment processor: CASH credits (v1) + Statement Store (v2), bearer-coin claims via host |
| t3rminal | 2026-06-08 | TypeScript | Static Next.js app on a `.dot` domain, backed by pallet-revive contract, metadata on Bulletin chain |
| mercado-community | 2026-06-08 | TypeScript | (no description) — marketplace ("mercado") app |
| localdot-community | 2026-06-08 | TypeScript | (no description) — local/"localdot" community app |

*Older but within 7 days (06-05/06-06):* t3rminal-OS-test, pocket-collectibles-webview, game-results-webview, w3spay-admin.
*Ignored:* `e2e-cli-moddable-*` auto-cleaned CI fixtures.

## Notable releases / activity
- **truapi → v0.3.0** (released 2026-06-03; repo pushed 06-09): host-side codegen, scheduled push notifications, Monaco-editor playground, design-system token alignment. (v0.1.0 = protocol v1.0 baseline, 05-15.) **No new protocol version**; still v1.0 transport.
- **w3s-payment-processor** (06-08): introduces a **v2 payment flow** (Statement Store) running in parallel with v1 CASH credits on Main + People chains.
- **t3rminal** actively pushed (06-09) alongside new **t3rminal-OS-test** scratch repo.

## Analyst take
The burst of `*-community` apps (mercado, localdot, playground, festival) plus a dedicated **identity-backend** and **w3s-payment-processor** suggests Parity is populating the "Products" layer of the Host + Products + TrUAPI super-app stack with reference mini-apps spanning commerce, identity, ticketing, and payments. *Hypothesis:* "W3S" appears to be the umbrella brand for this product suite, and the payment-processor's v1→v2 (CASH → Statement Store) shift hints the payments primitive is the most actively iterated. TrUAPI itself remains on protocol v1.0 — tooling (codegen, push, playground) is maturing faster than the wire protocol.

## Sources
- https://api.github.com/orgs/paritytech/repos?sort=created&direction=desc&per_page=40
- https://api.github.com/repos/paritytech/identity-backend-community/readme
- https://api.github.com/repos/paritytech/w3s-payment-processor/readme
- https://api.github.com/repos/paritytech/festival/readme
- https://api.github.com/repos/paritytech/w3s-architecture/readme
- https://api.github.com/repos/paritytech/t3rminal/readme
- https://api.github.com/repos/paritytech/truapi/releases
