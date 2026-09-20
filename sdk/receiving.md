---
label: Receiving Emails
icon: inbox
order: 30
---

# Receiving Emails

---

## Nothing to call

The sync engine keeps the ndk cache filled from the relays and revisits it on
its own. The client declares what the logged-in account needs and follows
logins and account switches. A freshly created client is already syncing.

Pull to refresh maps to one call:

```dart
await client.fetchRecent();
```

---

## Watching

```dart
client.onEmail.listen((email) {
  print('${email.from}: ${email.subject}');
});
```

`watch()` gives the full stream of `MailEvent`s, which is what a mailbox UI
listens to: `EmailReceived`, `LabelAdded`, `LabelRemoved`, `EmailDeleted`.
The narrow streams `onLabel`, `onTrash`, `onRead` and `onStarred` are there for
the common cases. `stopWatching()` ends them.

```dart
client.watchUnreadCount(folder: 'inbox').listen((count) => badge.value = count);
```

---

## Listing a mailbox

`getSummaries` is what a message list should call. It reads only the indexed
columns a row draws, so it never touches the stored MIME nor parses it.

```dart
final page = await client.getSummaries(folder: 'inbox', limit: 50, offset: 0);

for (final row in page.items) {
  print('${row.isRead ? ' ' : '*'} ${row.fromName ?? row.from}: ${row.subject}');
  print('  ${row.preview}');
}

if (page.hasMore) {
  // next page at offset: page.offset + page.items.length
}
```

A row carries `to`, `cc`, `bcc`, `date`, `folder`, `labels`, `attachmentRefs`,
`isPublic` and `isBridged`. For a native Nostr sender, `from` is `<npub>@nostr`
and the real name lives in the profile behind `senderPubkey`: resolve that
first, and fall back to `fromName ?? from`.

Filters: `folder`, `isRead`, `isStarred`, `hasAttachments`, `senderPubkey`,
`fromAddress`, `search`.

!!!warning Filtering by sender
A bridged email carries the bridge's pubkey, shared by every sender behind it,
so pass `fromAddress` as well for those rows. The address alone would let
anyone claim it.
!!!

```dart
final fromSender = await client.getSummaries(
  senderPubkey: row.senderPubkey,
  fromAddress: row.isBridged ? row.from : null,
);
```

---

## Opening one email

```dart
final email = await client.getEmail(row.id);
print(email!.body);
```

`Email` exposes `subject`, `body`, `textBody`, `htmlBody`, `date`, `from`,
`attachmentRefs`, and the parsed `mime` underneath.

Attachment bytes are loaded on demand, from the Blossom cache when they are
there and by rebuilding the original MIME when they are not:

```dart
final bytes = await client.getAttachmentBytes(email, email.attachmentRefs.first);
final eml = await client.getRawMimeText(email); // byte-exact original
```

---

## Folders and labels

Folders, read state and stars are NIP-32 labels, each in its own gift wrap.
The client keeps a local view and publishes the changes.

```dart
await client.markAsRead(email.id);
await client.star(email.id);
await client.moveToTrash(email.id);
await client.restoreFromTrash(email.id);
await client.moveToArchive(email.id);

await client.addLabel(email.id, 'folder:invoices');
await client.removeLabel(email.id, 'folder:invoices');
```

Convenience readers: `getInboxEmails`, `getSentEmails`, `getTrashedEmails`,
`getArchivedEmails`, `getStarredEmails`, `getTrashedEmailsOlderThan`,
`getUnreadCount`, plus the matching `*EmailIds` lists.

---

## Search

```dart
final results = await client.search('meeting', limit: 10);
```

Full-text search runs on the local store, across folders. For a scoped search,
pass `search:` to `getSummaries`.

---

## Deleting

```dart
await client.delete([email.id]);
```

One NIP-09 request covers the batch. It targets the gift wrap, which is the
event a relay actually holds, and it is signed by the pubkey the wrap is
addressed to. The labels attached to those emails go with them.

---

## When a wrap does not make it

Decryption can fail, a remote signer can refuse, a Blossom blob can be
unreachable. Those wraps are parked rather than dropped.

```dart
final failed = await client.getFailedGiftWraps();
for (final f in failed) {
  print('${f.event.id} stopped at ${f.progress.stage}: ${f.progress.failure}');
}

await client.retry(failed.first.event.id);
final count = await client.getFailedCount();
```

---

## Introspection

The wrap, the seal and the rumor behind an email stay available, which is what
lets a client show the real event or export it:

```dart
final wrap = await client.getGiftWrap(email.id);
final seal = await client.getSeal(email.id);
final rumor = await client.getRumor(email.id);
```
