<div align="center">

# MichkaMC — Mail &amp; Chat

**A secure, modern Android app that unites a full email client, end‑to‑end‑encrypted username‑only chat, calendar, to‑dos and contacts — with everything encrypted on the device.**

Source‑available · Non‑commercial · Android 8.0+ · Kotlin + Jetpack Compose

</div>

---

> ⚠️ **License:** MichkaMC is **source‑available, not open source**. It is free for **non‑commercial** use under the [PolyForm Noncommercial License 1.0.0](LICENSE). **Commercial use requires a paid license** — see [Licensing](#licensing).

## What it is

MichkaMC ("Michka Mail &amp; Chat") is a privacy‑first Android client that puts your email, private messaging, calendar, to‑dos and contacts in one app — and keeps all of it encrypted at rest with a key that only you can unlock. The chat is fully end‑to‑end encrypted and routes through a **zero‑knowledge relay** you can self‑host, so no server ever sees your messages.

No emojis anywhere — the UI uses clean vector icons only. Material 3, dark and light themes, 60 fps Compose UI.

## Features

### 📧 Email
- Multi‑account **IMAP / POP3 / SMTP** (up to 10 accounts), TLS/STARTTLS enforced with server‑identity checks.
- Real **autodiscovery**: provider autoconfig XML, Thunderbird ISPDB, and DNS SRV (RFC 6186/8314) over DNS‑over‑HTTPS, with a sensible last‑resort guess.
- **OAuth2 foundation** for Gmail / Outlook / Yahoo (AppAuth); falls back to password / app‑password until client IDs are configured.
- Collapsible **per‑account** navigation with nested folders (Inbox, Sent, Drafts, Spam, Trash, Archive, custom) and unread counts — no unified inbox, strong per‑account isolation.
- Message list (sender / subject / preview / date, unread dot, star, attachment icon) and a message reader that shows **plain text by default**, a per‑message **HTML toggle**, and **blocks remote images** until you tap "display images" (defeats tracking pixels).
- Compose / reply / reply‑all / forward, **attachments** (pick, send, download &amp; open; hard 100 MB cap per email), drafts, and IMAP `APPEND` of sent mail.
- New‑mail notifications with optional sound and ephemeral, never‑stored previews.

### 💬 Chat (end‑to‑end encrypted)
- **Identity is an on‑device keypair** — no phone number, no email, never registered. Your public key *is* your address (Session‑style), shared out‑of‑band via **QR code** or a `michkamc:` link.
- **Signal‑Protocol E2E**: X3DH + Double Ratchet, **re‑implemented from the public spec** on BouncyCastle primitives (X25519, Ed25519, HKDF, HMAC, AES‑256‑GCM). No `libsignal` dependency.
- **Zero‑knowledge relay** (self‑hostable — see below): forwards only opaque ciphertext, keyed by public id. It never holds a private key or plaintext.
- **Reliable delivery**: messages are held until the recipient acknowledges the exact envelope, redelivered on every reconnect, kept for a **72‑hour TTL**, with WebSocket keepalive and per‑sender **flood limiting (100 msg/min)**.
- Long‑press a conversation to **delete** it or **block** the sender; long‑press a message to delete it. Local chat history is encrypted on device.
- Your chosen display name travels **inside the encrypted body**, so contacts see your name without the relay ever learning it.

### 📅 Calendar · ✅ To‑do · 👤 Contacts
- Local **calendar** (month grid, per‑day events, all‑day toggle, start/end date &amp; time, location, notes).
- **To‑dos/notes** with due dates and done state.
- **Contacts**: manual CRUD, `.vcf` (vCard) import, optional dynamic import of email senders, and long‑press‑to‑email.
- All three live in their own app‑level database, independent of email — usable with **no account configured**.

### 🔒 Security &amp; privacy
- **Full‑database encryption** with SQLCipher — every database (mail, per‑account, app content, chat) is encrypted.
- One random **Master Data Key (MDK)** encrypts everything (databases + stored credentials + chat identity/sessions). Nothing sensitive is stored in the clear.
- **Two‑factor at rest**: the MDK is wrapped by `Keystore( PBKDF2(PIN) → MDK )` once a PIN is set — unlocking needs **both** the correct PIN *and* the device's hardware‑backed Keystore. A stolen file can't be brute‑forced off‑device.
- **App‑lock PIN**: 6‑digit, on‑screen keypad (no system keyboard), **5 tries/minute → 15‑minute lockout** (persisted), weak PINs rejected, masked entry. Changing the PIN requires the current one.
- **Mandatory once you have data**: the moment you create your first account, chat identity, or app data, a PIN is required — and from then on your databases can only be decrypted with it.
- Credentials never in plaintext; no analytics or telemetry.

### 🌍 Other
- **11 languages** in the picker (English wired; others fall back to English until translated).
- Light / dark themes with a brand palette.

## Security architecture (in brief)

| Layer | How it works |
|---|---|
| **At rest** | Random 256‑bit Master Data Key → SQLCipher passphrase for all DBs + AES‑GCM for stored secrets. |
| **Key protection** | MDK wrapped device‑bound (Keystore) until a PIN exists, then strict `Keystore(PBKDF2(PIN)→MDK)`. Wrong PIN fails the authenticated decrypt. Changing the PIN re‑wraps only the key — no data is re‑encrypted. |
| **Chat E2E** | X3DH key agreement + Double Ratchet, per‑message keys, from‑scratch on BouncyCastle. |
| **Transport** | Zero‑knowledge relay: opaque ciphertext + public‑key‑keyed prekey directory + store‑and‑forward + live WebSocket. |

> Trade‑off of "strict": if you forget your PIN, the data is unrecoverable by design. A portable, password‑protected **encrypted backup/restore** is on the roadmap as the recovery path.

## Tech stack

Kotlin · Jetpack Compose + Material 3 · MVVM + unidirectional data flow · Hilt (DI) · Coroutines + Flow · Room + **SQLCipher** · Jakarta Mail (Angus Mail) for IMAP/POP3/SMTP · OkHttp · AppAuth · BouncyCastle · ZXing (QR) · DataStore · Android Keystore. Relay: **Ktor** (Netty, WebSockets, kotlinx.serialization).

`minSdk 26 (Android 8.0)` · `compileSdk 35` · Gradle (Kotlin DSL) + version catalog.

## Repository layout

```
app/            Android app (Compose UI, data layer, DI)
server-chat/    Self-hostable zero-knowledge chat relay (Ktor)
Relay/          Relay deployment kit (systemd/nginx/Apache configs + scripts)
LICENSE                 PolyForm Noncommercial 1.0.0
THIRD_PARTY_NOTICES.md  Bundled third-party components + their licenses
```

## Build &amp; run

Requirements: a **JDK** (not just a JRE) — the Android Studio JBR works well.

```bash
# Debug APK
./gradlew :app:assembleDebug

# Run the relay locally (listens on 0.0.0.0:8080; HOST=127.0.0.1 for loopback)
./gradlew :server-chat:run
```

On Windows/PowerShell, point Gradle at a JDK first if it can't find a compiler:

```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"
```

To enable OAuth2 for Gmail/Outlook/Yahoo, register your own OAuth client IDs and add them as `buildConfigField`s (see the OAuth notes in the source). Without them the app uses password / app‑password auth.

## Self‑hosting the chat relay

The `server-chat` module is a thin, self‑hostable **zero‑knowledge relay** that stores only public keys and opaque ciphertext. It can run on a small Debian VPS **without Docker** (systemd + nginx/Apache + certbot). Build a distribution and deploy it with the scripts in `Relay/`:

```bash
./gradlew :server-chat:installDist   # or :server-chat:distTar
```

The Android client points at a relay via `BuildConfig.RELAY_BASE_URL`. See `server-chat/README.md` and `Relay/` for the API and deployment runbook.

## Status

Actively developed. Built and working today: accounts &amp; secure storage, mail send/receive with attachments, E2E chat over a live relay, calendar/to‑do/contacts, full at‑rest encryption + app‑lock PIN, i18n. On the roadmap: encrypted backup/restore (key portability), background push, an optional `@username` name service, and rules/spam filtering.

## Licensing

MichkaMC is **source‑available, not open source**.

- **Non‑commercial use is free** under the [PolyForm Noncommercial License 1.0.0](LICENSE) — read, copy, run, modify and share for any non‑commercial purpose.
- **Commercial use requires a separate paid license** from the copyright holder. Contact: **https://michkamc.org**.
- Third‑party libraries remain under their own licenses (all permissive or linkable) — see [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md). Their notices must be preserved when redistributing.

Because this is a copyleft‑incompatible, non‑OSI license, please do not submit contributions derived from GPL/copyleft projects. If contributions are accepted in future, a CLA will be required so commercial dual‑licensing remains possible.

## Author

**Michael DALLA RIVA** — https://michkamc.org

Copyright © 2026 Michael DALLA RIVA. All rights reserved.
