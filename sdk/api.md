---
label: API Reference
icon: code
order: 10
---

# API Reference

The surface of `nostr_mail` 3.2.x. Full dartdoc lives on
[pub.dev](https://pub.dev/documentation/nostr_mail/latest/).

---

## NostrMailClient.create

```dart
static Future<NostrMailClient> create({
  required Ndk ndk,
  required NostrMailDatabase database,
  required Database db,
  required BlossomCache blossomCache,
  required SyncEngine syncEngine,
  List<String>? defaultDmRelays,
  List<String>? defaultBlossomServers,
  Map<String, String>? nip05Overrides,
  OfflineBroadcast? broadcastQueue,
  OfflineBlossomUpload? blossomUploadQueue,
  String? schedulerDvm,
  List<String>? schedulerDvmReadRelays,
})
```

| Parameter | Description |
|-----------|-------------|
| `ndk` | Relays, signer and event cache, with one account logged in |
| `database` | The drift mail store |
| `db` | The sembast database shared by the queues and the scheduler |
| `blossomCache` | Local blob store for large emails and attachments |
| `syncEngine` | Keeps the ndk cache filled. Started by the client, never stopped by it |
| `defaultDmRelays` | Fallback DM relays for an account without a kind 10050 list |
| `defaultBlossomServers` | Fallback Blossom servers for an account without a kind 10063 list |
| `nip05Overrides` | Pin a NIP-05 name to a pubkey, bypassing the lookup |
| `broadcastQueue` / `blossomUploadQueue` | Share the queues your app already runs. Passing one makes its lifecycle yours |
| `schedulerDvm` | Default Scheduler DVM pubkey for `scheduleEmail` |

---

## Sending

```dart
Future<void> send({
  required List<Recipient> to,
  List<Recipient> cc = const [],
  List<Recipient> bcc = const [],
  required String subject,
  required String body,
  MailAddress? from,
  String? htmlBody,
  bool keepCopy = true,
  bool signRumor = false,
  bool isPublic = false,
})

Future<void> sendMime(
  MimeMessage message, {
  required List<Recipient> to,
  List<Recipient> cc = const [],
  List<Recipient> bcc = const [],
  bool keepCopy = true,
  bool signRumor = false,
  bool isPublic = false,
  String? mailFrom,
  Future<void> Function(Nip01Event event, List<String> relays)? beforePublish,
})
```

---

## Scheduling

| Method | Description |
|--------|-------------|
| `scheduleEmail({..., required DateTime at, String? dvmPubkey})` | Hand a prepared email to a Scheduler DVM |
| `scheduleMime(message, {..., required DateTime at})` | Same, with a message you built |
| `getScheduledEmails()` | Not-yet-sent emails, soonest first |
| `watchScheduledEmails()` | Reactive version, re-emitting on DVM feedback |
| `cancelScheduledEmail(packageId)` | Cancel before the send time |
| `getScheduledMime(packageId)` | Rebuild the full editable MIME for a composer |
| `resyncScheduledEmails()` | One-shot network resync |
| `startScheduling()` / `stopScheduling()` | Live DVM feedback and multi-device sync |

---

## Reading

| Method | Returns | Description |
|--------|---------|-------------|
| `getSummaries({folder, isRead, isStarred, hasAttachments, senderPubkey, fromAddress, search, limit, offset})` | `PaginatedResult<EmailSummary>` | The call a message list makes |
| `getEmails({limit, offset})` | `List<Email>` | Full messages, newest first |
| `getEmail(id)` | `Email?` | One full message |
| `openEmail(...)` | `Email?` | Fetch an email that is not local yet |
| `search(query, {limit, offset})` | `List<Email>` | Full-text search across folders |
| `getInboxEmails()` / `getSentEmails()` / `getTrashedEmails()` / `getArchivedEmails()` / `getStarredEmails()` | `List<Email>` | Folder readers |
| `getTrashedEmailsOlderThan(duration)` | `List<Email>` | For an auto-empty policy |
| `getUnreadCount({folder})` / `watchUnreadCount({folder})` | `int` / `Stream<int>` | Badge counts |
| `getRawMimeText(email)` / `getRawMime(email)` | `String?` / `MimeMessage?` | The byte-exact original, for `.eml` export or quoting |
| `getAttachmentBytes(email, ref)` | `Uint8List?` | Attachment bytes, from cache or rebuilt |
| `getGiftWrap(id)` / `getSeal(id)` / `getRumor(id)` | `Nip01Event?` | NIP-59 introspection |

---

## Labels

| Method | Description |
|--------|-------------|
| `addLabel(emailId, label)` / `removeLabel(emailId, label)` | Any label in the `mail` namespace |
| `getLabels(emailId)` / `hasLabel(emailId, label)` | Read them back |
| `markAsRead` / `markAsUnread` / `star` / `unstar` | State labels |
| `moveToTrash` / `restoreFromTrash` / `moveToArchive` / `restoreFromArchive` | Folder labels |
| `isRead` / `isStarred` / `isTrashed` / `isArchived` | Predicates |
| `getTrashedEmailIds` / `getArchivedEmailIds` / `getStarredEmailIds` / `getReadEmailIds` | Id lists |

---

## Sync and events

| Member | Description |
|--------|-------------|
| `fetchRecent()` | Go to the relays now, whatever the coverage |
| `watch()` | `Stream<MailEvent>`: received, label added or removed, deleted |
| `onEmail` | `Stream<Email>` of new mail |
| `onLabel` / `onTrash` / `onRead` / `onStarred` | Narrow streams |
| `stopWatching()` | End the subscriptions |
| `getFailedGiftWraps()` / `getFailedCount()` / `retry(eventId)` | Wraps that never reached the store |
| `delete(ids)` | One NIP-09 request for a batch |
| `repost(emailEvent)` | Republish a public email |

---

## Settings

| Method | Description |
|--------|-------------|
| `cachedPrivateSettings({pubkey})` | The in-memory copy, no I/O |
| `getLocalPrivateSettings({pubkey})` | The stored copy |
| `fetchPrivateSettings({pubkey})` | Go to the relays |
| `getPrivateSettings({pubkey, timeout})` | `NdkDataResponse`: local first, then the network |
| `setPrivateSettings(settings, {pubkey})` | Replace |
| `updatePrivateSettings({signature, bridges, identities, clear*})` | Merge, with explicit clears |

`PrivateSettings` holds `signature`, `bridges` and `identities`, the last being
RFC 5322 strings usable directly as a `From` header. The first identity is the
default From address, also exposed as `defaultAddress`.

---

## Lifecycle

| Method | Description |
|--------|-------------|
| `clearLocalAccountData({required pubkey})` | Drop one account's local records and pending work |
| `clearAllLocalData()` | Drop every account's |
| `dispose()` | Stop the workers, dispose the owned queues |

---

## Models

**`Recipient`** is sealed: `NostrRecipient` (a `pubkey` plus a display
`mailAddress`) or `SmtpRecipient` (an address relayed by a bridge).
`resolveRecipient({to, ndk, nip05Overrides})` classifies a raw address.

**`Email`**: `id`, `senderPubkey`, `recipientPubkey`, `isPublic`, `isBridged`,
`createdAt`, `subject`, `date`, `from`, `body`, `textBody`, `htmlBody`,
`attachmentRefs`, `mime`, and the Blossom fields `blossomHash`,
`decryptionKey`, `decryptionNonce`.

**`EmailSummary`**: `id`, `senderPubkey`, `from`, `fromName`, `to`, `cc`, `bcc`,
`subject`, `preview`, `date`, `folder`, `isRead`, `isStarred`, `labels`,
`attachmentRefs`, `isPublic`, `isBridged`.

**`PaginatedResult<T>`**: `items`, `total`, `offset`, `hasMore`.

**`ScheduledEmail`**: the package id, the send time and a
`ScheduledEmailStatus`.

**`FailedGiftWrap`**: the `event` and its `progress` (`GiftWrapStage`,
`GiftWrapFailure`, `attempts`).

---

## Exceptions

| Exception | Meaning |
|-----------|---------|
| `NostrMailException` | Base class, for example no account configured |
| `RecipientResolutionException` | The lookup answered, and the answer was no |
| `BridgeResolutionException` | No bridge for that domain |
| `NetworkRequiredException` | The call never got an answer. Carries `operation` |
| `EmailParseException` | The MIME could not be parsed |
| `RelayException` | A relay refused or failed |

---

## Constants

| Constant | Value |
|----------|-------|
| `emailKind` | `1301` |
| `giftWrapKind` | `1059` |
| `labelKind` | `1985` |
| `deletionRequestKind` | `5` |
| `dmRelayListKind` | `10050` |
| `relayListKind` | `10002` |
| `blossomServerListKind` | `10063` |
| `appSettingsKind` | `30078` |
| `labelNamespace` | `mail` |
| `publicSettingsDTag` | `nostr-mail/settings` |
| `privateSettingsDTag` | `nostr-mail/settings/private` |
| `maxInlineSize` | `32768` |

`recommendedDmRelays` and `recommendedBlossomServers` are the defaults used for
an account with no list of its own.
