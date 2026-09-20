---
label: Getting Started
icon: rocket
order: 80
---

# Getting Started

Pick the path that matches what you want to do.

---

## Choose Your Path

+++ I want to send/receive emails
Use **Nmail**, the reference client, or build your own app with an **SDK**.

[:icon-device-mobile: Nmail](/client/) | [:icon-package: SDKs](/sdk/)
+++ I want to run my own bridge
Deploy a **bridge** to connect legacy email with Nostr.

[:icon-server: Bridge Setup](/bridge/)
+++ I want to understand the protocol
Read the **protocol specification** to learn how it works.

[:icon-book: Protocol](/protocol/)
+++

---

## For End Users

[!button icon="mail" text="Open Nmail" target="blank"](https://app.nostrmail.org)

1. Open the web app, or install it from
   [ZapStore](https://zapstore.dev/apps/app.nostrmail.client) or the
   [releases page](https://github.com/nogringo/nostr-mail-client/releases/latest).
2. Generate a key, or sign in with a signer you already use.
3. Back up your key, then start writing.

Your address is `<npub>@nostr` out of the box. A readable address like
`alice@example.com` comes from a NIP-05 provider, and is also what lets people
on ordinary email reach you.

---

## For Developers

### Dart and Flutter

```bash
dart pub add nostr_mail
```

```dart
final client = await NostrMailClient.create(
  ndk: ndk,
  database: database,
  db: db,
  blossomCache: blossomCache,
  syncEngine: syncEngine,
);

await client.send(
  to: [NostrRecipient.fromPubkey(bobPubkey)],
  subject: 'Hello from Nostr!',
  body: 'This email was sent over the Nostr protocol.',
);

client.onEmail.listen((email) {
  print('New email: ${email.mime.decodeSubject()}');
});
```

[Full SDK documentation :icon-arrow-right:](/sdk/)

### JavaScript and TypeScript

```bash
npm install nostr-mail
```

```javascript
const client = new NostrMailClient(secretKey);

await client.sendEmail({
  to: 'npub1...',
  subject: 'Hello from Nostr',
  text: 'Hey! This is a private email sent over Nostr.',
});
```

[JavaScript SDK :icon-arrow-right:](/sdk/javascript/)

---

## For Bridge Operators

### Prerequisites

- A server with Docker, or Node.js and Dart
- A domain with MX records configured
- A Nostr private key for the bridge

### Quick Start with Docker

```bash
git clone https://github.com/nogringo/nostr-mail-bridge
cd nostr-mail-bridge
cp .env.example .env
# Edit .env with your configuration
docker compose up -d
```

[Full bridge setup :icon-arrow-right:](/bridge/)

---

## Email Address Formats

| Format | Example | Best For |
|--------|---------|----------|
| NIP-05 | `alice@example.com` | Human-readable, reachable from legacy email |
| npub@domain | `npub1...@bridge.com` | Sovereign identity with legacy compatibility |
| npub@nostr | `npub1...@nostr` | Nostr-to-Nostr communication |

!!!warning Note
The `npub@nostr` format is not routable by legacy email servers. Use it only for
Nostr-to-Nostr communication.
!!!

---

## Next Steps

- [:icon-book: Understand the protocol](/protocol/)
- [:icon-server: Set up a bridge](/bridge/)
- [:icon-package: Build with an SDK](/sdk/)
- [:icon-device-mobile: Use Nmail](/client/)
