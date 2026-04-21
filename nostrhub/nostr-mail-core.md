# Nostr Email Protocol

## Goal

Remove gatekeepers from email. Use Nostr as transport instead of SMTP between users.

## Event Kind

### Kind 1301: Email

```json
{
  "kind": 1301,
  "pubkey": "<sender pubkey>",
  "content": "<RFC 2822 email>"
}
```

The content is a standard email. Nostr is just the delivery mechanism.

## Sending

There are 2 kinds of users, those using nostr and the others. If the recipient is not on nostr we need to send the email to a bridge that will forward the email to the recipient legacy inbox.

Nostr emails uses NIP-59 gift wraps for privacy. It's similar to NIP-17.

1. Create the event kind 1301
2. Gift wrap it
3. Send it to recipient DMs relays

### Sending to a nostr user

```json
{
  "kind": 1301,
  "pubkey": "<sender pubkey>",
  "content": "<RFC 2822 email>"
}
```

#### Example

```json
{
  "kind": 1301,
  "pubkey": "alice pubkey",
  "content": "From: npub1alice...@nostr\nTo: npub1bob...@nostr\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Bob, how are you?"
}
```

### Sending to a non nostr user

Sending to a non nostr user require using a bridge.

```json
{
  "kind": 1301,
  "pubkey": "<sender pubkey>",
  "tags": [
    ["mail-from", "<sender_email>"],
    ["rcpt-to", "<recipient_email>"]
  ],
  "content": "<RFC 2822 email>"
}
```

- You can get the `bridge pubkey` by resolving `_smtp@bridge_domain` with NIP-05.
- Multiple `rcpt-to` tags MAY be used for CC/BCC recipients.

#### Example

```json
{
  "kind": 1301,
  "pubkey": "alice pubkey",
  "tags": [
    ["mail-from", "npub1alice...@bridge.com"],
    ["rcpt-to", "bob@example.com"]
  ],
  "content": "From: npub1alice...@bridge.com\nTo: bob@example.com\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Bob, how are you?"
}
```

## Authentication
 
By default, the kind 1301 rumor is unsigned, providing deniability.
 
If the sender wants to prove authorship to third parties, they MAY sign the rumor. A signed rumor is a fully valid Nostr event with a `sig` field. This is useful in professional or legal contexts where the sender needs to prove they wrote an email.
 
```json
{
  "kind": 1301,
  "pubkey": "alice pubkey",
  "content": "From: npub1alice...@nostr\nTo: npub1bob...@nostr\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Bob, how are you?",
  "sig": "..."
}
```
 
Since a signed rumor is a valid Nostr event, it can be published to relays or reposted by anyone. This allows third parties to independently verify that the sender wrote the email, without requiring any trust in the recipient.

## Public Emails

Emails sent to public entities MAY be sent without gift wrap, published directly to relays as a plain kind 1301 event. In this case the event MUST be signed, providing a public and verifiable proof that the sender wrote the email.

This is useful for communications with public entities (governments, companies) where transparency is required.
