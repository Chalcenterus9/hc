# hc.py

Minimalist [hack.chat](https://github.com/hack-chat) client with end-to-end encryption.

PWA, runs in any browser on any device. No servers, no accounts, no trackers — just a static page and a WebSocket to hack.chat.

## What is this

hc.py is a client for the anonymous chat [hack.chat](https://hack.chat) that adds an encryption layer on top of the open protocol. Both parties enter the same secret phrase — a room address and encryption key are deterministically derived from it. The hack.chat server only sees ciphertext.

The project comes as a PWA (`index.html`) web client, installable via "Add to Home Screen"

## Features

- **E2E encryption** — AES-256-GCM, HKDF-derived key, hourly rotation (UTC, ±1 hour window)
- **PBKDF2 rooms** — room name is derived via PBKDF2 (500k iterations), brute-forcing the secret phrase from the room hash is impractical
- **Vault** — encrypted room storage on device (AES-GCM, master password, PBKDF2 100k)
- **Ghost mode** — no localStorage writes whatsoever
- **Fingerprint** — 4-emoji key fingerprint for peer verification
- **Markdown** — `**bold**`, `*italic*`, `` `code` ``, `~~strike~~`, `__underline__`, `==highlight==` with colors
- **Mentions** — `@nick` is highlighted, double beep and push notification
- **Themes** — 4 built-in themes (Default, Matrix, Nostalgic, Nord Light)
- **Native commands** — `/whisper`, `/me`, `/nick`, `/move` and others are sent as plaintext, bypassing encryption
- **Reconnect** — automatic reconnection with exponential backoff on connection loss
- **PWA** — offline cache, Service Worker updates, iOS safe area support

## What this is NOT

- No registration or accounts
- No server-side component (beyond hack.chat itself)
- No message history on the server
- No file transfer (yet)
- No multi-chat / tabs (yet)
- No analytics, trackers, or ads

## Installation

Open in a browser and add to home screen:

**→ [chalcenterus9.github.io/hc](https://chalcenterus9.github.io/hc/)**

iOS: Safari → Share → "Add to Home Screen".
Android: Chrome → menu → "Install app".

The app works offline after the first visit (except for the chat itself, which requires a network connection).

## Threat model

### What it protects against

**The hack.chat server cannot read message contents.** All text is encrypted client-side with AES-256-GCM; the server only receives `enc:period_id:base64`. The encryption key never leaves the device.

**The room name does not reveal the secret phrase.** The room address is a PBKDF2 output with 500,000 iterations. Brute-forcing short phrases from the room hash takes significant time even on a GPU.

**Key rotation limits the compromise window.** The encryption key changes every hour (UTC). Compromising a single period key does not reveal messages from other periods.

**Ghost mode leaves no traces on the device.** In this mode localStorage is not used — no nick, no server, no vault.

### What it does NOT protect against

**The secret phrase is the only secret.** If the phrase is weak (dictionary word, short), an attacker can crack it offline and gain access to all messages. Use 4+ random words.

**No forward secrecy.** All participants use the same key derived from the phrase. Compromising the phrase reveals all messages — past and future. Hourly rotation only helps if an individual period key is compromised, not the phrase itself.

**No participant authentication.** Anyone who knows the phrase can join the room and send messages under any nick. The fingerprint confirms key equality, not the identity of the other party.

**Metadata is visible to the server.** The server knows: IP addresses, connection times, traffic volume, number of participants in a room. Content — no, but the fact of communication — yes.

**Native commands are not encrypted.** `/whisper`, `/me`, `/nick` and other server commands are sent as plaintext per the hack.chat protocol.

**Vault is protected by a master password.** If the master password is weak, an attacker with device access can decrypt the room list. Secret phrases are stored in the vault — this is a deliberate trade-off for convenience.

**Client code is loaded from the hosting provider.** On every page load you trust GitHub Pages (or another host) to serve unmodified code. Self-hosting or Service Worker pinning reduces this risk.

### Recommendations

- Secret phrase: 4+ random words, not from a dictionary, do not reuse
- For additional anonymity: Tor Browser
- To verify your peer: compare fingerprints over a separate channel
- For maximum stealth: Ghost mode + Tor Browser + do not save the phrase

## Stack

- Plain HTML/CSS/JS, no frameworks or build steps
- Web Crypto API (AES-GCM, HKDF, PBKDF2)
- Web Audio API for notifications
- Service Worker for PWA and offline cache
- WebSocket to the hack.chat server

## Roadmap

- [ ] Personalization settings (font size, emote commands, clickable links)
- [ ] Multi-chat — tabs with separate connections
- [ ] P2P file transfer via WebRTC
- [ ] Custom protocol with message history and voice messages

## Related projects

- [hack.chat](https://github.com/hack-chat) — the server and original web client that hc.py is built on top of

## License

MIT — [Chalcenterus9](https://github.com/Chalcenterus9), made with help of Claude AI
