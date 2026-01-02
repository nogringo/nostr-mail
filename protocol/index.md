---
label: Protocol
icon: book
order: 90
---

# Nostr Mail Protocol

The Nostr Mail protocol defines how emails are transmitted over the Nostr network using Kind 1301 events.

---

## Goal

Remove gatekeepers from email. Use Nostr as transport instead of SMTP between users.

---

## Event Kind

### Kind 1301: Email

```json
{
  "kind": 1301,
  "pubkey": "<sender_npub>",
  "tags": [
    ["p", "<recipient_npub>"]
  ],
  "content": "<RFC 2822 email>"
}
```

The content is a standard email (RFC 2822). Nostr is just the delivery mechanism.

---

## Example

### Nostr-to-Nostr Email

```json
{
  "kind": 1301,
  "pubkey": "npub1alice...",
  "tags": [
    ["p", "npub1bob..."]
  ],
  "content": "From: alice@bridge.mail\nTo: bob@bridge.mail\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Bob, how are you?"
}
```

---

## Sending to Legacy Email

When the recipient has no npub (e.g., `alice@gmail.com`), send to a bridge:

```json
{
  "kind": 1301,
  "pubkey": "npub1bob...",
  "tags": [
    ["p", "<bridge_npub>"]
  ],
  "content": "From: bob@bridge.mail\nTo: alice@gmail.com\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Alice!"
}
```

### How it works

1. `p` tag = bridge's npub (the bridge receives the event)
2. `To:` header in content = legacy recipient
3. Bridge extracts `To:` and sends via SMTP

---

## Receiving from Legacy Email

When `alice@gmail.com` sends to `bob@bridge.mail`:

```json
{
  "kind": 1301,
  "pubkey": "<bridge_npub>",
  "tags": [
    ["p", "npub1bob..."]
  ],
  "content": "From: alice@gmail.com\nTo: bob@bridge.mail\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Bob!"
}
```

### How it works

1. Bridge receives SMTP email
2. Bridge looks up npub for `bob@bridge.mail`
3. Bridge creates event and publishes to Bob's NIP-17 DM relays

---

## Privacy

Emails are gift-wrapped using [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) so relays only see:
- Recipient pubkey
- Encrypted blob

The actual email content is never visible to relays.

---

## Bridge Role

The bridge converts transport only:

| Direction | From | To | Process |
|-----------|------|-----|---------|
| Inbound | SMTP | Nostr event | Parses `To:` to find npub |
| Outbound | Nostr event | SMTP | Parses `To:` to find legacy address |

---

## Bridge Discovery

Use NIP-05 to find a bridge's npub:

```http
GET https://bridge.mail/.well-known/nostr.json?name=_smtp

{
  "names": {
    "_smtp": "<bridge_npub_hex>"
  }
}
```

### Discovery Flow

1. User selects a bridge (e.g., `bridge.mail`)
2. Client queries `_smtp@bridge.mail` via NIP-05
3. Client gets bridge npub
4. Client sends kind 1301 event with `p` = bridge npub

---

## Relay Discovery

Use [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) DM relays (kind 10050) for message delivery.

The client should:
1. Look up recipient's DM relay list (kind 10050)
2. Publish the gift-wrapped event to those relays
