---
label: Local Storage
icon: database
order: 20
---

# Local Storage

The SDK stores mail locally so the app reads from disk, not from the relays.
Four stores are involved, and you own all of them.

| Store | What it holds |
|-------|---------------|
| `NostrMailDatabase` (drift/SQLite) | Emails, summaries, labels, gift wraps, settings, tombstones |
| `sembast` database | The broadcast queue, the Blossom upload queue, the scheduler, the sync engine |
| `BlossomCache` | Large-email blobs and attachment bytes |
| ndk cache | Raw events, owned by ndk itself |

---

## Native

```dart
import 'dart:io';
import 'package:drift/native.dart';
import 'package:sembast/sembast_io.dart' hide Filter;

final database = NostrMailDatabase(NativeDatabase(File('nostr_mail.sqlite')));
final db = await databaseFactoryIo.openDatabase('emails.db');
final blossomCache = await IdbBlossomCache.open(factory: idbFactorySembastIo);
```

In Flutter, put both files under `getApplicationDocumentsDirectory()`.

---

## Web

Copy `sqlite3.wasm` (from the [sqlite3.dart](https://github.com/simolus3/sqlite3.dart/releases)
release matching your resolved `sqlite3` version) and `drift_worker.js` (from
the [drift](https://github.com/simolus3/drift/releases) release matching
`drift`) into `web/`, then open the store through a worker:

```dart
import 'package:drift/wasm.dart';

final result = await WasmDatabase.open(
  databaseName: 'nostr_mail',
  sqlite3Uri: Uri.parse('sqlite3.wasm'),
  driftWorkerUri: Uri.parse('drift_worker.js'),
);
final database = NostrMailDatabase(result.resolvedExecutor);
```

Use `idbFactoryBrowser` for the Blossom cache on web.

Check `result.chosenImplementation`. Drift stores the file in OPFS when the
browser allows it: `opfsShared` needs no headers on Chrome and Firefox,
`opfsLocks` needs the page to be cross-origin isolated, which is what Safari
falls back to. Otherwise it falls back to IndexedDB, which works but keeps the
file image in memory.

!!!warning Where the memory actually goes
A `SembastCacheManager` given to ndk holds every raw event in memory. With a
large mailbox, that cache, not the mail store, is the one to watch.
!!!

---

## Full-text search

The drift store indexes emails with SQLite FTS5, which is what makes
`client.search(...)` and the `search:` filter of `getSummaries` local and
instant.

---

## Clearing

```dart
// One account: its emails, labels, wraps, settings, and its pending work.
await client.clearLocalAccountData(pubkey: pubkey);

// Every account.
await client.clearAllLocalData();
```

Both leave the ndk cache alone, so a later sync rebuilds the mail from it.
Clear that cache too to forget an account entirely.

A queue you passed to `create` stays yours: clear it with
`OfflineBroadcast.clearLocalAccountData(pubkey:)` and
`OfflineBlossomUpload.clearLocalAccountData(pubkey:)`.

!!!danger Do not clear an account that is sending
A broadcast overlapping the clear can re-create its record.
!!!

---

## Shutting down

```dart
await client.dispose();
await database.close();
await db.close();
```

`dispose` stops the background workers and disposes the queues it owns, waiting
for any attempt in flight. Queues you passed in are yours to dispose, and so are
the Blossom cache and the sync engine.
