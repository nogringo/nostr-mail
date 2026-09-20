---
label: SDK
icon: package
order: 60
expanded: true
---

# Dart SDK

A Dart SDK for sending and receiving emails over the Nostr protocol using NIP-59
gift-wrapped messages.

[![Pub Version](https://img.shields.io/pub/v/nostr_mail)](https://pub.dev/packages/nostr_mail)

There is also a [JavaScript SDK](javascript.md) for Node and the browser.

---

## Installation

```bash
dart pub add nostr_mail
```

Or add to your `pubspec.yaml`:

```yaml
dependencies:
  nostr_mail: ^3.2.1
```

---

## Features

- Send to Nostr users, to legacy addresses through a bridge, or both at once
- NIP-59 gift wrapping, with signed rumors and public emails when you want them
- Attachments of any size, stored encrypted on Blossom
- Local store on drift (SQLite) with full-text search
- Folders, read state and stars as NIP-32 labels
- Settings synced across devices with NIP-78
- Scheduled sending through a Scheduler DVM
- Offline-first: queued sends and uploads survive a restart

---

## What you provide

The client does not own its infrastructure. You build it and hand it over, so it
can share what your app already runs.

| Dependency | Role |
|------------|------|
| [`ndk`](https://pub.dev/packages/ndk) | Relays, signing, the event cache. One account logged in. |
| `NostrMailDatabase` | The mail store, drift over SQLite. `NativeDatabase` on native, `WasmDatabase` on web. |
| `sembast` database | Shared by the broadcast queue, the Blossom upload queue and the scheduler. |
| [`SyncEngine`](https://pub.dev/packages/sync_engine_shim_for_ndk) | Keeps the ndk cache filled from the relays. Yours to own: the client starts it, never stops it. |
| [`BlossomCache`](https://pub.dev/packages/blossom_cache) | Local blob store for large emails and attachments. |

---

## Quick Start

```dart
import 'dart:io';

import 'package:drift/native.dart';
import 'package:ndk/ndk.dart';
import 'package:nostr_mail/nostr_mail.dart';
import 'package:sembast/sembast_io.dart' hide Filter;
import 'package:sync_engine_shim_for_ndk/sync_engine_shim_for_ndk.dart';

void main() async {
  final ndk = Ndk(NdkConfig(
    cache: MemCacheManager(),
    eventVerifier: Bip340EventVerifier(),
  ));
  final keyPair = Bip340.generatePrivateKey();
  ndk.accounts.loginPrivateKey(
    pubkey: keyPair.publicKey,
    privkey: keyPair.privateKey!,
  );

  final database = NostrMailDatabase(NativeDatabase(File('nostr_mail.sqlite')));
  final db = await databaseFactoryIo.openDatabase('emails.db');
  final blossomCache = await IdbBlossomCache.open(factory: idbFactorySembastIo);
  final syncEngine = SyncEngine(ndk, db: db);

  final client = await NostrMailClient.create(
    ndk: ndk,
    database: database,
    db: db,
    blossomCache: blossomCache,
    syncEngine: syncEngine,
  );

  await client.send(
    to: [await resolveRecipient(to: 'bob@example.com', ndk: ndk)],
    subject: 'Hello from Nostr!',
    body: 'This email was sent over the Nostr protocol.',
  );

  client.onEmail.listen((email) {
    print('New email: ${email.mime.decodeSubject()}');
  });
}
```

!!!warning Remote signers
With a remote signer, give ndk a cache manager that persists
(`SembastCacheManager`). ndk keeps the plaintext it decrypted there, so
approvals already granted are not asked for again.
!!!

---

## Quick Navigation

||| [:icon-mail: Sending Emails](sending.md)
Recipients, transports, scheduling, public emails
||| [:icon-inbox: Receiving Emails](receiving.md)
Sync, mailbox listing, labels and folders
||| [:icon-database: Local Storage](storage.md)
The drift store, Blossom cache, clearing an account
||| [:icon-code: API Reference](api.md)
The client surface, models and exceptions
||| [:icon-logo-github: JavaScript SDK](javascript.md)
The same protocol from Node and the browser
|||
