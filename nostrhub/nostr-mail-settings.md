# Nostr Mail Settings

This document defines the protocol for storing and synchronizing user settings across devices in Nostr Mail using NIP-78 application-specific data.

## Overview

User settings are stored as replaceable parameterized events (kind 30078). This enables seamless synchronization across multiple devices while maintaining privacy through encryption.

## Event Kind

### Kind 30078: Application-Specific Data

Settings use NIP-78 replaceable parameterized events with the `d` tag for namespacing.

## Public Settings

Public settings are stored unencrypted and can be read by anyone.

```json
{
  "kind": 30078,
  "pubkey": "<user_pubkey>",
  "tags": [["d", "nostr-mail/settings"]],
  "content": "{\"dm_copy\": true, \"prefer_nostr\": true}"
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `dm_copy` | boolean | Request bridges to send a DM copy of incoming emails |
| `prefer_nostr` | boolean | Deliver email for this key's NIP-05 addresses over Nostr rather than SMTP. |

When updating either settings event, a client MUST keep the fields of `content` it does not change.

## Transport Preference

For a recipient given as an email address `name@domain`:

1. Resolve it through NIP-05. If it does not resolve to a pubkey, deliver over SMTP.
2. Fetch the pubkey's public settings from its NIP-65 write relays.
3. If `prefer_nostr` is `true`, deliver over Nostr. Otherwise, deliver over SMTP.

A failed lookup counts as `false`.

## Private Settings

Private settings MUST be encrypted to self using NIP-44.

```json
{
  "kind": 30078,
  "pubkey": "<user_pubkey>",
  "tags": [["d", "nostr-mail/settings/private"]],
  "content": "<nip44_encrypted_json>"
}
```

### Decrypted Content

After decryption, the content contains:

| Field | Type | Description |
|-------|------|-------------|
| `signature` | string | Email signature appended to outgoing emails |
| `bridges` | string[] | List of preferred bridge domains |
| `identities` | string[] | List of user-defined "From" identities in RFC 2822 format |

### Identities

Identities are user-defined "From" addresses stored as RFC 5322 formatted strings. Each entry can be used directly in the `From:` header without any transformation.

**Format examples:**
- `"Alice Real <npub1abc...@nostr.mail>"`: name + address
- `"npub1abc...@bridge.com"`: address only (no name)
- `"Pseudo <alice@example.com>"`: custom name + legacy email

**Behavior:**
- If `identities` is empty or absent, clients SHOULD auto-generate available addresses from `npub@nostr` and configured bridges
- The **first identity** (index 0) is the default "From" address

### Example (Decrypted)

```json
{
  "signature": "Sent via Nostr Mail",
  "bridges": ["nostr.mail", "bridge.example.com"],
  "identities": [
    "Alice Real <npub1abc...@nostr.mail>",
    "<npub1abc...@bridge.com>",
    "Pseudo <alice@example.com>"
  ]
}
```

### Example (Encrypted Event)

```json
{
  "kind": 30078,
  "tags": [["d", "nostr-mail/settings/private"]],
  "content": "<nip44_encrypted_blob>"
}
```
