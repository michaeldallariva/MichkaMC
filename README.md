<div align="center">

# MichkaMC (Mail and Chat)

**A secure Android app that unites a full email client, end-to-end-encrypted username-only chat, calendar, to-dos and contacts, with everything encrypted on the device.**

Source-available | Non-commercial | Android 8.0+ | Kotlin and Jetpack Compose

</div>

---

> **License:** MichkaMC is source-available, not open source. It is free for non-commercial use under the [PolyForm Noncommercial License 1.0.0](LICENSE). Commercial use requires a paid license. See [Licensing](#licensing).

## What it is

MichkaMC (Michka Mail and Chat) is a privacy-first Android client that puts your email, private messaging, calendar, to-dos and contacts in one app, and keeps all of it encrypted at rest with a key only you can unlock. The chat is fully end-to-end encrypted and routes through a zero-knowledge relay you can self-host, so no server ever sees your messages. The interface is Material 3 with light and dark themes and a smooth Compose UI.

## Features

### Email

- Multi-account IMAP, POP3 and SMTP (up to 10 accounts), with TLS/STARTTLS enforced and server identity checked.
- Real autodiscovery: provider autoconfig, Thunderbird ISPDB, and DNS SRV over DNS-over-HTTPS, with a sensible fallback.
- OAuth2 sign-in for Gmail, Outlook and Yahoo, plus password and app-password authentication.
- Collapsible per-account navigation with nested folders (Inbox, Sent, Drafts, Spam, Trash, Archive and custom folders) and unread counts, with strong per-account isolation.
- Message reader that shows plain text by default, a per-message HTML toggle, and blocks remote images until you allow them (defeats tracking pixels).
- Compose, reply, reply-all and forward, with attachments (send, download and open; up to 100 MB per email), drafts, and a copy of sent mail filed to your Sent folder.
- New-mail notifications with an optional sound and ephemeral, never-stored previews.

### Chat (end-to-end encrypted)

- Your identity is an on-device keypair: no phone number, no email, never registered. Your public key is your address, shared out-of-band via a QR code or a `michkamc:` link.
- Signal-Protocol encryption (X3DH and Double Ratchet) built on modern primitives (X25519, Ed25519, HKDF, HMAC, AES-256-GCM), giving every message its own key.
- A zero-knowledge relay (self-hostable, see below) that forwards only opaque ciphertext and never holds a private key or plaintext.
- Reliable delivery: messages are held until the recipient acknowledges the exact envelope, redelivered on every reconnect, retained for up to 72 hours, with WebSocket keepalive and per-sender flood limiting.
- Encrypted local chat history, delete a conversation or a single message, and block a sender.
- Your chosen display name travels inside the encrypted body, so contacts see your name without the relay ever learning it.

### Calendar, To-do and Contacts

- Local calendar with a month grid, per-day events, all-day toggle, start and end date and time, location and notes.
- To-dos and notes with due dates and a done state.
- Contacts with manual create, edit and delete, vCard (.vcf) import, and long-press to email a contact.
- All three work with no email account configured.

### Security and privacy

- Full-database encryption with SQLCipher: every database (mail, per-account, app content and chat) is encrypted.
- One random Master Data Key encrypts everything (databases, stored credentials, and chat identity and sessions). Nothing sensitive is stored in the clear.
- Two-factor protection at rest: once a PIN is set, the master key is unlocked only with both the correct PIN and the device's hardware-backed Keystore, so a copied file cannot be brute-forced off the device.
- App-lock PIN: 6-digit, entered on an on-screen keypad, with a 5-tries-per-minute limit and a 15-minute lockout, weak PINs rejected, and masked entry. Changing the PIN requires the current one.
- A PIN is required as soon as you create your first account, chat identity or data, and from then on your databases can only be decrypted with it.
- No analytics and no telemetry.

### Interface

- Available in multiple languages.
- Light and dark themes.

## Security architecture

| Layer | How it works |
|---|---|
| At rest | A random 256-bit Master Data Key is the SQLCipher passphrase for all databases and the key for stored secrets. |
| Key protection | The master key is wrapped device-bound (Keystore) until a PIN exists, then strictly with both the PIN (PBKDF2) and the Keystore. A wrong PIN fails the authenticated decryption. Changing the PIN re-wraps only the key, so no data is re-encrypted. |
| Chat encryption | X3DH key agreement plus a Double Ratchet, with a fresh key per message. |
| Transport | A zero-knowledge relay: opaque ciphertext, a public-key-keyed prekey directory, store-and-forward, and live WebSocket delivery. |

If you forget your PIN, the data cannot be recovered. This is by design for strict at-rest encryption.

## Technology

Kotlin, Jetpack Compose with Material 3, MVVM with unidirectional data flow, Hilt for dependency injection, Coroutines and Flow, Room with SQLCipher, Jakarta Mail for IMAP/POP3/SMTP, OkHttp, AppAuth, BouncyCastle, ZXing for QR codes, DataStore, and the Android Keystore. The relay is built with Ktor (Netty, WebSockets, kotlinx.serialization).

Minimum Android 8.0 (API 26), compiled against API 35, built with Gradle (Kotlin DSL) and a version catalog.

## Repository layout

```
app/            Android app (Compose UI, data layer, dependency injection)
server-chat/    Self-hostable zero-knowledge chat relay (Ktor)
Relay/          Relay deployment kit (service configs and scripts)
LICENSE                 PolyForm Noncommercial 1.0.0
THIRD_PARTY_NOTICES.md  Bundled third-party components and their licenses
```

## Build and run

A JDK is required (the Android Studio JBR works well).

```bash
# Debug APK
./gradlew :app:assembleDebug

# Run the relay locally (listens on 0.0.0.0:8080; set HOST=127.0.0.1 for loopback)
./gradlew :server-chat:run
```

On Windows PowerShell, point Gradle at a JDK first if it cannot find a compiler:

```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"
```

To enable OAuth2 for Gmail, Outlook or Yahoo, add your own OAuth client IDs as build config fields; otherwise the app uses password or app-password authentication.

## Self-hosting the chat relay

The `server-chat` module is a thin, self-hostable zero-knowledge relay that stores only public keys and opaque ciphertext. It runs on a small Debian server without Docker (systemd with nginx or Apache and certbot). Build a distribution and deploy it with the scripts in `Relay/`:

```bash
./gradlew :server-chat:installDist   # or :server-chat:distTar
```

The Android client points at a relay via its build configuration. See `server-chat/README.md` and `Relay/` for the API and the deployment runbook.

## Licensing

MichkaMC is source-available, not open source.

- Non-commercial use is free under the [PolyForm Noncommercial License 1.0.0](LICENSE): read, copy, run, modify and share for any non-commercial purpose.
- Commercial use requires a separate paid license from the copyright holder. Contact: https://michkamc.org
- Third-party libraries remain under their own licenses; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md). Their notices must be preserved when redistributing.

## Author

Michael DALLA RIVA, https://michkamc.org

Copyright (c) 2026 Michael DALLA RIVA. All rights reserved.

<img width="469" height="1060" alt="Image" src="https://github.com/user-attachments/assets/61cc07d0-b804-48b0-b354-862606b7a48a" />

<img width="473" height="1058" alt="Image" src="https://github.com/user-attachments/assets/e5fea576-9135-4172-b5a8-7655781c438d" />

<img width="477" height="1066" alt="Image" src="https://github.com/user-attachments/assets/52679605-2180-4cd3-a2d0-65fecfd8c3f6" />

<img width="482" height="1062" alt="Image" src="https://github.com/user-attachments/assets/a7ed4e10-4b6d-4987-899a-a537b0d33d30" />


