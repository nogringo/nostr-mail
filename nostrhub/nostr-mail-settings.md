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
  "content": "{\"dm_copy\": true}"
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `dm_copy` | boolean | Request bridges to send a DM copy of incoming emails |

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
| `default_address` | string | Default "From" address (e.g. `npub1...@bridge.com`) |
| `signature` | string | Email signature appended to outgoing emails |
| `bridges` | string[] | List of preferred bridge domains |

### Example (Decrypted)

```json
{
  "default_address": "npub1abc...@nostr.mail",
  "signature": "Sent via Nostr Mail",
  "bridges": ["nostr.mail", "bridge.example.com"]
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
