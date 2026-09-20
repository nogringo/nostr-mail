---
label: Sending Emails
icon: mail
order: 40
---

# Sending Emails

---

## Recipients carry their transport

`send` takes typed recipients, not raw strings. The transport is decided before
the send starts, so a NIP-05 lookup that fails halfway can never misroute a
Nostr recipient through a bridge.

```dart
// Native Nostr, from a pubkey. The display address becomes <npub>@nostr.
NostrRecipient.fromPubkey(bobPubkey);

// Native Nostr, keeping the address the user typed.
NostrRecipient(pubkey: bobPubkey, mailAddress: MailAddress(null, 'bob@example.com'));

// Legacy, relayed through your SMTP bridge.
SmtpRecipient('bob@gmail.com');
```

To classify an address you only have as text, use `resolveRecipient`:

```dart
final recipient = await resolveRecipient(to: 'bob@example.com', ndk: ndk);
```

| Input | Result |
|-------|--------|
| `npub1...`, hex, `npub1...@domain` | `NostrRecipient`, no network call |
| `user@domain` with a NIP-05 hit | `NostrRecipient` |
| `user@domain` with no such name | `SmtpRecipient` |
| Network error or malformed answer | throws, rather than guessing |

---

## Basic send

```dart
await client.send(
  to: [NostrRecipient.fromPubkey(bobPubkey)],
  cc: [SmtpRecipient('carol@example.com')],
  subject: 'Hello!',
  body: 'This is the email body.',
);
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `to` | `List<Recipient>` | required | Primary recipients |
| `cc` / `bcc` | `List<Recipient>` | `[]` | Copies. Bcc recipients each get their own wrap |
| `subject` | `String` | required | Subject |
| `body` | `String` | required | Plain text body |
| `from` | `MailAddress?` | identity | The From header, defaults to your first identity |
| `htmlBody` | `String?` | none | HTML alternative |
| `keepCopy` | `bool` | `true` | Wrap a copy to yourself, which is what fills Sent |
| `signRumor` | `bool` | `false` | Sign the rumor to prove authorship |
| `isPublic` | `bool` | `false` | Publish a signed, unwrapped event instead |

---

## Sending a prepared MIME message

When you build the message yourself, attachments included:

```dart
await client.sendMime(
  message,
  to: [SmtpRecipient('bob@example.com')],
  mailFrom: 'npub1alice...@bridge.com',
);
```

`beforePublish` is called for every outgoing event once its relays are resolved
and just before it is queued, which is where a client hooks progress reporting.

---

## Signed and public emails

`signRumor: true` turns the rumor into a complete Nostr event. The recipient can
show it to a third party and that third party can verify it, without trusting
anyone. Deniability is the default precisely because this is not always wanted.

`isPublic: true` publishes the signed event to the relays instead of wrapping
it. Bcc recipients still get a wrap, carrying a `public-ref` tag that points at
the public event and the relays where it can be fetched. This is the shape for
writing to a public entity where transparency is the point.

---

## Scheduling

A [Scheduler DVM](https://openspecs.uid.ovh/spec/npub1kg4sdvz3l4fr99n2jdz2vdxe2mpacva87hkdetv76ywacsfq5leqquw5te/scheduler-dvm)
holds the prepared email and publishes it at the requested time.

```dart
final scheduled = await client.scheduleEmail(
  to: [NostrRecipient.fromPubkey(bobPubkey)],
  subject: 'Monday reminder',
  body: 'See you at 10.',
  at: DateTime.now().add(const Duration(days: 3)),
);

await client.cancelScheduledEmail(scheduled.packageId);
```

Scheduling, listing and cancelling are local-first and work offline. Call
`startScheduling()` to also receive live DVM feedback and multi-device updates,
and watch the list with `watchScheduledEmails()`. The email lands in Sent
through the normal sync once the DVM actually publishes it.

Set the DVM once in `NostrMailClient.create(schedulerDvm: ...)`, or per call
with `dvmPubkey`.

---

## What happens under the hood

1. Build the RFC 2822 message.
2. Create the kind 1301 rumor, with `email-id` and, for a bridged recipient,
   `mail-from` and `rcpt-to`.
3. Move the MIME to Blossom, encrypted, when it exceeds the inline threshold
   (32 KB).
4. Gift wrap once per recipient.
5. Hand each wrap to the offline broadcast queue, which resolves the destination
   relays and keeps retrying across restarts.

---

## Error handling

```dart
try {
  await client.send(to: [recipient], subject: 'Test', body: 'Test');
} on NetworkRequiredException catch (e) {
  print('Offline during ${e.operation}, ask the user to reconnect');
} on RecipientResolutionException catch (e) {
  print('No such recipient: ${e.message}');
} on NostrMailException catch (e) {
  print('Error: ${e.message}');
}
```

`NetworkRequiredException` means the lookup never got an answer.
`RecipientResolutionException` means it got one, and the answer was no.
