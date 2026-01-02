---
label: API Reference
icon: code
order: 10
---

# API Reference

Complete API documentation for the Nostr Mail SDK.

---

## NostrMailClient

Main client for sending and receiving emails.

### Constructor

```dart
NostrMailClient({
  required Ndk ndk,
  required Database db,
})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `ndk` | `Ndk` | Nostr Development Kit instance |
| `db` | `Database` | Sembast database instance |

---

### Methods

#### send()

Send an email to a recipient.

```dart
Future<void> send({
  required String to,
  required String subject,
  required String body,
  String? from,
  String? htmlBody,
  bool keepCopy = true,
})
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `to` | `String` | Yes | Recipient (npub, hex, or email) |
| `subject` | `String` | Yes | Email subject |
| `body` | `String` | Yes | Plain text body |
| `from` | `String?` | Sometimes | Required for legacy emails |
| `htmlBody` | `String?` | No | Optional HTML content |
| `keepCopy` | `bool` | No | Send copy to self (default: true) |

**Throws:**
- `NostrMailException` - Account not configured
- `RecipientResolutionException` - Cannot resolve recipient

---

#### watchInbox()

Stream of incoming emails in real-time.

```dart
Stream<Email> watchInbox()
```

**Returns:** `Stream<Email>` - Yields emails as they arrive

**Throws:**
- `NostrMailException` - Account not configured

---

#### sync()

Fetch and store historical emails from relays.

```dart
Future<void> sync()
```

**Throws:**
- `NostrMailException` - Account not configured

---

#### getEmails()

Get cached emails from local storage.

```dart
Future<List<Email>> getEmails({
  int? limit,
  int? offset,
})
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | `int?` | Maximum emails to return |
| `offset` | `int?` | Skip first N emails |

**Returns:** `List<Email>` - Sorted by date (newest first)

---

#### getEmail()

Get a single email by ID.

```dart
Future<Email?> getEmail(String id)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `String` | Event ID |

**Returns:** `Email?` - The email or null if not found

---

#### delete()

Delete an email from local storage.

```dart
Future<void> delete(String id)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `String` | Event ID to delete |

---

## Email

Email model class.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | `String` | Unique event ID |
| `from` | `String` | Sender address |
| `to` | `String` | Recipient address |
| `subject` | `String` | Email subject |
| `body` | `String` | Plain text body |
| `date` | `DateTime` | Email timestamp |
| `senderPubkey` | `String` | Nostr pubkey of sender |
| `rawContent` | `String` | Original RFC 2822 content |

### Methods

#### toJson()

```dart
Map<String, dynamic> toJson()
```

#### fromJson()

```dart
factory Email.fromJson(Map<String, dynamic> json)
```

---

## Exceptions

### NostrMailException

Base exception for SDK errors.

```dart
class NostrMailException implements Exception {
  final String message;
  NostrMailException(this.message);
}
```

### RecipientResolutionException

Thrown when recipient cannot be resolved.

```dart
class RecipientResolutionException extends NostrMailException {
  final String address;
  RecipientResolutionException(this.address);
}
```

---

## Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `_emailKind` | `1301` | Nostr event kind for emails |

---

## Dependencies

```yaml
dependencies:
  ndk: ^0.6.1-dev.4
  enough_mail: ^2.1.7
  sembast: ^3.8.6
  http: ^1.6.0
```
