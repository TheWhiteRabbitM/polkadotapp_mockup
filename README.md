# 🐇 Polkadot super-app — Web3 Summit demo (community recreation)

> Couldn't make it to **Web3 Summit**? It was demoed live, on stage, for the people in the room.
> So I rebuilt it for everyone else — a hands-on, **as-close-to-real-as-possible** recreation of the Polkadot self-custodial super-app, pieced together from Parity's **open-source repos** and the **public app build**.
>
> **▶ Try it:** open **`index.html`** (or the GitHub Pages link once deployed).

---

## What it is

A single-file, **100% offline** web recreation of the app. Pick a username, **check in**, chat & call, pay at the **W3S Pay merchant terminal**, scan a QR, play the **DIM2 gesture game**, earn your **membership** — across all **5 themes**.

It runs entirely in the browser: **no blockchain, no network, no wallet, no accounts, no servers, no cookies, no analytics.** Every balance, payment, identity and check-in is **simulated**, stored **nowhere**, and gone the moment you refresh.

## Deploy to GitHub Pages

1. Create a repo and push these files (keep the folder structure — `index.html` + `assets/`).
2. **Settings → Pages → Build and deployment → Source: `Deploy from a branch` → `main` / `/ (root)`** → **Save**.
3. Live at `https://<user>.github.io/<repo>/`.

`.nojekyll` is included so Pages serves files as-is. No build step. (Fonts load from Google Fonts; everything else is local.)
For correct social-share previews, edit the `og:image` / `twitter:image` meta in `index.html` to your absolute Pages URL.

## What's in the repo

| File | |
|---|---|
| **`index.html`** | the demo (main page) |
| `assets/apk/` | real image assets extracted from the public APK (Polkadot logo, collectibles art, prizes pattern, scanner frame, proof-of-ink) |
| `assets/screenshots/` | reference screenshots |
| `concept.html` | earlier black-&-white cyberpunk concept landing |
| `Polkadot-SuperApp-Deck.html` · `Polkadot-SuperApp-Video.html` | analyst slide deck / cinematic |
| `Parity-*.md` | the analysis write-ups this was built from |

## How it was built

- **Structure & UI** read from the open-source iOS Host `paritytech/polkadot-ios-community` (tab bar Chat·Pocket·Browse·Settings, the Pocket card stack, DSButton metrics, verbatim strings, the 5 themes).
- **Visual assets** extracted from the public APK (`PolkadotUI` drawables) for fidelity.
- **Features simulated** from what the app actually ships — calls (WebRTC), DIM2 gesture game + voting, Proof-of-Ink tattoo flow, Mob Rule, vouchers/bearer-coins, PoUD allowances, personhood score, passkey backup & force-reclaim, contact requests, notifications, username upgrade, QR scanner, send→sign→outcome.

## ⚠️ Disclaimer

This is an **independent, community recreation** — **not** the official app, and **not affiliated with or endorsed by Parity / Polkadot / Web3 Foundation**. It is **offline and simulated**: no real accounts, no real payments, **no real value**. Nothing is stored. Trademarks belong to their owners. **Not financial advice.**

*🐇 follow the white rabbit.*
