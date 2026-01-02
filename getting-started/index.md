---
label: Getting Started
icon: rocket
order: 80
---

# Getting Started

This guide will help you get started with Nostr Mail depending on your use case.

---

## Choose Your Path

+++ I want to send/receive emails
Use the **Flutter Client** or build your own app with the **Dart SDK**.

[:icon-device-mobile: Client Setup](/client/) | [:icon-package: SDK Documentation](/sdk/)
+++ I want to run my own bridge
Deploy the **Bridge** to connect legacy email with Nostr.

[:icon-server: Bridge Setup](/bridge/)
+++ I want to understand the protocol
Read the **Protocol Specification** to learn how it works.

[:icon-book: Protocol](/protocol/)
+++

---

## For End Users

### Use the Flutter Client

[!button icon="globe" text="Try Online" target="blank"](https://nogringo.github.io/nostr-mail-client)

1. Open the web client or download the app
2. Generate or import your Nostr keys
3. Start sending and receiving emails!

---

## For Developers

### Using the Dart SDK

Install the package:

```bash
dart pub add nostr_mail
```

Basic usage:

```dart
import 'package:nostr_mail/nostr_mail.dart';
import 'package:ndk/ndk.dart';
import 'package:sembast/sembast_io.dart';

// Initialize NDK with your keys
final ndk = Ndk(NdkConfig(
  eventSigner: Bip340EventSigner(privateKey: 'your-hex-key'),
  bootstrapRelays: ['wss://relay.damus.io', 'wss://nos.lol'],
));

// Open local database
final db = await databaseFactoryIo.openDatabase('emails.db');

// Create client
final client = NostrMailClient(ndk: ndk, db: db);

// Send an email
await client.send(
  to: 'recipient@example.com',
  subject: 'Hello from Nostr!',
  body: 'This email was sent over the Nostr protocol.',
  from: 'me@bridge.mail',
);

// Listen for incoming emails
await for (final email in client.watchInbox()) {
  print('New email from: ${email.from}');
  print('Subject: ${email.subject}');
}
```

[Full SDK Documentation :icon-arrow-right:](/sdk/)

---

## For Bridge Operators

### Prerequisites

- A server with Docker (recommended) or Node.js
- A domain with MX records configured
- Nostr private key (hex format)

### Quick Start with Docker

```bash
git clone https://github.com/nogringo/nostr-mail-bridge
cd nostr-mail-bridge
cp .env.example .env
# Edit .env with your configuration
docker-compose up -d
```

[Full Bridge Setup :icon-arrow-right:](/bridge/)

---

## Email Address Formats

Nostr Mail supports multiple address formats:

| Format | Example | Best For |
|--------|---------|----------|
| NIP-05 | `alice@bridge.com` | User-friendly addresses |
| npub@domain | `npub1...@bridge.com` | Sovereign identity with legacy compatibility |
| npub@nostr | `npub1...@nostr` | Nostr-native communication |

!!!warning Note
The `npub@nostr` format is not routable by legacy email servers. Use it only for Nostr-to-Nostr communication.
!!!

---

## Next Steps

- [:icon-book: Understand the Protocol](/protocol/)
- [:icon-server: Set up a Bridge](/bridge/)
- [:icon-package: Build with the SDK](/sdk/)
- [:icon-device-mobile: Try the Client](/client/)
