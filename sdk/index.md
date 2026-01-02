---
label: SDK
icon: package
order: 60
expanded: true
---

# Dart SDK

A Dart SDK for sending and receiving emails over the Nostr protocol using NIP-59 gift-wrapped messages.

[![Pub Version](https://img.shields.io/pub/v/nostr_mail)](https://pub.dev/packages/nostr_mail)

---

## Installation

```bash
dart pub add nostr_mail
```

Or add to your `pubspec.yaml`:

```yaml
dependencies:
  nostr_mail: ^1.1.0
```

---

## Features

- Send and receive emails over Nostr
- NIP-59 gift-wrapped encryption
- Automatic bridge resolution for legacy emails
- Local email storage with Sembast
- Real-time inbox streaming
- Historical email sync

---

## Quick Start

```dart
import 'package:nostr_mail/nostr_mail.dart';
import 'package:ndk/ndk.dart';
import 'package:sembast/sembast_io.dart';

void main() async {
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
    from: 'me@mybridge.com',  // Required for legacy emails
  );

  // Listen for new emails
  await for (final email in client.watchInbox()) {
    print('New email from: ${email.from}');
    print('Subject: ${email.subject}');
  }
}
```

---

## Dependencies

| Package | Description |
|---------|-------------|
| `ndk` | Nostr Development Kit for Dart |
| `enough_mail` | RFC 2822 email parsing |
| `sembast` | Local NoSQL database |
| `http` | HTTP client for NIP-05 resolution |

---

## Quick Navigation

||| [:icon-mail: Sending Emails](sending.md)
How to send emails to Nostr and legacy addresses
||| [:icon-inbox: Receiving Emails](receiving.md)
Watch your inbox and sync historical emails
||| [:icon-database: Local Storage](storage.md)
Email persistence with Sembast
||| [:icon-code: API Reference](api.md)
Complete API documentation
|||
