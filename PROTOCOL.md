# Nostr Email Protocol

## Goal

Remove gatekeepers from email. Use Nostr as transport instead of SMTP between users.

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

The content is a standard email. Nostr is just the delivery mechanism.

## Example

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

## Sending to a Legacy Email (e.g. alice@gmail.com)

When the recipient has no npub, send to a bridge:

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

1. `p` tag = bridge's npub (the bridge receives the event)
2. `To:` header in content = legacy recipient
3. Bridge extracts `To:` and sends via SMTP

## Receiving from a Legacy Email

When alice@gmail.com sends to bob@bridge.mail:

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

1. Bridge receives SMTP email
2. Bridge looks up npub for bob@bridge.mail
3. Bridge creates event and publishes to Bob's NIP-65 inbox relays

## Privacy

Emails are gift-wrapped (NIP-59) so relays only see recipient pubkey + encrypted blob.

## Bridge

Converts transport only:
- Inbound: SMTP → Nostr event (parses `To:` to find npub)
- Outbound: Nostr event → SMTP (parses `To:` to find legacy address)

## Bridge Discovery

Use NIP-05 to find a bridge's npub:

```
GET https://bridge.mail/.well-known/nostr.json?name=_smtp

{
  "names": {
    "_smtp": "<bridge_npub_hex>"
  }
}
```

When sending to `alice@gmail.com`:
1. User selects a bridge (e.g. `bridge.mail`)
2. Client queries `_smtp@bridge.mail` via NIP-05
3. Client gets bridge npub
4. Client sends kind 1301 event with `p` = bridge npub

## Relay Discovery

Use NIP-65 inbox relays.
