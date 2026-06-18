# Parity / paritytech Org Watch — Digest

**Date of scan:** 2026-06-18
**Baseline:** Previous digest 2026-06-10. "New" = created after that run (06-10 → 06-18).

## New repositories since last check

| Name | Created | Lang | One-line description |
|------|---------|------|----------------------|
| **flow-funder** 🆕 | 2026-06-15 | Rust | Bot that watches finalized People-chain `PeopleLite::attest` calls and reserves each attested username as a `dotns` name on Asset Hub (needs a funded signer for fees). |
| **vocabulario-community** 🆕 | 2026-06-13 | JavaScript | Per-wallet, on-chain language-learning dictionary; quizzes + publish/share dictionaries via Bulletin chain, runs as a product in the Polkadot browser host. |
| **polkadot-desktop-community** | 2026-06-10 | TypeScript | "Polkadot Desktop prototype" — desktop port of the Polkadot user-agent. |
| **polkadot-ios-community** | 2026-06-10 | Swift | "Polkadot iOS user-agent prototype". |

*Already covered on 06-10 (created 06-09/06-10):* identity-backend-community, polkadot-app-deploy, polkadot-android-community, playground-app-community, w3s-architecture.
*Ignored:* nine `e2e-cli-moddable-*` auto-cleaned CI fixtures (06-10 → 06-17).

## Notable releases / activity
- **truapi → v0.3.1** (released 2026-06-17): fixes only — corrected **`HostPaymentTopUpError` variant ordering on the wire protocol**, explorer import paths, CI/release tagging. **No new protocol version** (still v1.0 transport; 0.3.0 was 06-03).
- **polkadot-desktop-community** is actively pushed (last push 06-18 06:09), the freshest of the user-agent platform ports (now spanning android / ios / desktop).
- **identity-backend-community** & **playground-app-community** still receiving pushes (06-17), indicating ongoing platform iteration.

## Analyst take
The Polkadot user-agent is now being forked into **android / ios / desktop** prototypes, signaling the "Host" layer is moving toward multi-platform reach. *Hypothesis:* **flow-funder** + the truapi `HostPayment` fix suggest the identity primitive (`dotns` usernames via People-chain attestations) and a host-mediated payment/top-up flow are the actively-hardened plumbing beneath the "Products" layer, while new products (vocabulario) keep landing as thin on-chain mini-apps on Bulletin. TrUAPI itself stays on protocol v1.0 — only its tooling and payment error-handling are changing.

## Sources
- https://api.github.com/orgs/paritytech/repos?sort=created&direction=desc&per_page=40
- https://api.github.com/repos/paritytech/flow-funder/readme
- https://api.github.com/repos/paritytech/vocabulario-community/readme
- https://api.github.com/repos/paritytech/truapi/releases
