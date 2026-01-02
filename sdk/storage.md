---
label: Local Storage
icon: database
order: 20
---

# Local Storage

The SDK uses Sembast for local email persistence.

---

## Setup

### Dart (Native)

```dart
import 'package:sembast/sembast_io.dart';

final db = await databaseFactoryIo.openDatabase('emails.db');
final client = NostrMailClient(ndk: ndk, db: db);
```

### Flutter (Mobile/Desktop)

```dart
import 'package:path_provider/path_provider.dart';
import 'package:sembast/sembast_io.dart';

final dir = await getApplicationDocumentsDirectory();
final dbPath = '${dir.path}/emails.db';
final db = await databaseFactoryIo.openDatabase(dbPath);
final client = NostrMailClient(ndk: ndk, db: db);
```

### Flutter Web

```dart
import 'package:sembast_web/sembast_web.dart';

final db = await databaseFactoryWeb.openDatabase('emails');
final client = NostrMailClient(ndk: ndk, db: db);
```

---

## Storage Structure

The SDK manages two stores:

| Store | Purpose |
|-------|---------|
| `emails` | Parsed email records |
| `processed` | Processed event IDs (deduplication) |

---

## Email Store Schema

```json
{
  "id": "event-id",
  "from": "sender@example.com",
  "to": "recipient@example.com",
  "subject": "Hello",
  "body": "Email body text",
  "date": "2024-12-28T12:00:00.000Z",
  "senderPubkey": "abc123...",
  "rawContent": "From: sender@example.com\nTo: ..."
}
```

---

## Operations

### Save Email

Handled automatically by `sync()` and `watchInbox()`:

```dart
await _store.saveEmail(email);
```

### Get Emails

```dart
// All emails (sorted by date, newest first)
final emails = await client.getEmails();

// With pagination
final page = await client.getEmails(limit: 20, offset: 0);
```

### Get Single Email

```dart
final email = await client.getEmail('event-id');
```

### Delete Email

```dart
await client.delete('event-id');
```

---

## Deduplication

The processed store prevents reprocessing:

```dart
// Check if already processed
if (await _store.isProcessed(eventId)) {
  return; // Skip
}

// Mark as processed
await _store.markProcessed(eventId);
```

---

## Database Management

### Clear All Data

```dart
await db.close();
// Delete the database file
```

### Backup

The database is a single file that can be copied for backup.

---

## Cross-Platform Considerations

| Platform | Factory | Path |
|----------|---------|------|
| Mobile | `sembast_io` | `getApplicationDocumentsDirectory()` |
| Desktop | `sembast_io` | `getApplicationDocumentsDirectory()` |
| Web | `sembast_web` | IndexedDB (browser) |

---

## Example: Platform-Aware Initialization

```dart
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:sembast/sembast.dart';

Future<Database> openEmailDatabase() async {
  if (kIsWeb) {
    return await databaseFactoryWeb.openDatabase('emails');
  } else {
    final dir = await getApplicationDocumentsDirectory();
    return await databaseFactoryIo.openDatabase('${dir.path}/emails.db');
  }
}
```
