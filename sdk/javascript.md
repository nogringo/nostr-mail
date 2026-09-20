---
label: JavaScript SDK
icon: code-square
order: 5
---

# JavaScript SDK

A JavaScript library for Nostr Mail, for Node and the browser.

[![npm](https://img.shields.io/npm/v/nostr-mail)](https://www.npmjs.com/package/nostr-mail)

It covers the same protocol as the [Dart SDK](index.md) with a smaller surface:
send, receive, label, delete. There is no local store and no scheduler. Source:
[nogringo/nostr-mail-js](https://github.com/nogringo/nostr-mail-js).

---

## Installation

```bash
npm install nostr-mail
```

---

## Initialize

```javascript
import { NostrMailClient } from 'nostr-mail';
import { generateSecretKey } from 'nostr-tools/pure';

const secretKey = generateSecretKey();
const client = new NostrMailClient(secretKey);
```

The second argument takes either a relay list or an options object, and the
third a list of default Blossom servers. `addRelay(url)` and `removeRelay(url)`
adjust the set afterwards.

---

## Send

`sendEmail` wraps a copy to you as well by default, which is what makes the
message show up in Sent.

```javascript
await client.sendEmail({
  to: 'npub1...', // or 'bob@example.com'
  subject: 'Hello from Nostr',
  text: 'Hey! This is a private email sent over Nostr.',
  html: '<b>Hey!</b> This is a private email sent over Nostr.',
  selfCopy: true,
});
```

Anything above roughly 60 KB is encrypted with AES-GCM, uploaded to Blossom,
and referenced from the event, so attachments are not a special case.

---

## Receive

```javascript
const stop = client.onEmail((email) => {
  console.log('New email:', email.subject);
});

const emails = await client.listEmails();
emails.forEach((email) => console.log(`${email.folder}: ${email.subject}`));
```

`fromGiftWrap(event)` decodes a wrap you fetched yourself, and
`getRecipientRelays(to)` resolves where a recipient reads.

---

## Labels and folders

Read state and folders are NIP-32 labels, the same ones the Dart SDK and Nmail
write.

```javascript
await client.addLabel(email.id, 'state:read');
await client.addLabel(email.id, 'folder:done');
await client.addLabel(email.id, 'flag:starred');
```

---

## Delete

```javascript
await client.deleteEmail(email, 'Reason for deletion');
```

This deletes the gift wrap from the relays, removes the labels that pointed at
it, and cleans up the Blossom blob. `deleteEvent(eventId, reason, kind)` is the
lower-level form.

Call `close()` when you are done to release the relay connections.
