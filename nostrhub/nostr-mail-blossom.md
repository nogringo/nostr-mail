# Nostr Mail with Blossom Storage

## Overview

This specification defines how to handle large emails (MIME > 65KB) by storing encrypted MIME messages on Blossom servers. Small emails remain stored inline in the kind 1301 event content.

## Motivation

NIP-44 encryption has a hard limit of **65535 bytes** for plaintext messages.

Traditional emails can easily exceed this limit. This spec provides a hybrid approach:

- **Small emails (< 60KB)**: MIME stored inline in kind 1301 `content` field
- **Large emails (≥ 60KB)**: MIME encrypted with AES-GCM and stored on Blossom, kind 1301 contains reference + decryption key

This allows Nostr Mail to support emails of any size.

## Event Kind 1301: Email

### Inline MIME (small emails < 65KB)

```json
{
  "kind": 1301,
  "pubkey": "<sender_npub>",
  "tags": [
    ["p", "<recipient_npub>"],
    ["subject", "Hello"],
    ["from", "alice@nostr.mail"],
    ["to", "bob@example.com"],
    ["date", "1735401600"]
  ],
  "content": "From: alice@nostr.mail\nTo: bob@example.com\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Bob!"
}
```

### Blossom MIME (large emails ≥ 65KB)

```json
{
  "kind": 1301,
  "pubkey": "<sender_npub>",
  "tags": [
    ["p", "<recipient_npub>"],
    ["encryption-algorithm", "aes-gcm"],
    ["decryption-key", "<base64_key>"],
    ["decryption-nonce", "<base64_nonce>"],
    ["x", "<sha256_hash>"],
    ["subject", "Hello"],
    ["from", "alice@nostr.mail"],
    ["to", "bob@example.com"],
    ["date", "1735401600"]
  ],
  "content": ""
}
```

### Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `p` | Yes | Recipient pubkey (or bridge pubkey) |
| `encryption-algorithm` | Conditional | Encryption algorithm `aes-gcm` (only for Blossom MIME) |
| `decryption-key` | Conditional | The decryption key (only for Blossom MIME) |
| `decryption-nonce` | Conditional | The decryption nonce (only for Blossom MIME) |
| `x` | Conditional | SHA-256 hex of encrypted MIME (only for Blossom MIME) |
| `subject` | No | Email subject |
| `from` | No | From address |
| `to` | No | To address |
| `date` | No | Email timestamp (Unix epoch) |

### Detection

Clients detect the storage mode:

| Condition | Mode | Action |
|-----------|------|--------|
| `content` non-empty | Inline MIME | Parse content directly |
| `content` empty, `x` tag exists | Blossom MIME | Download from Blossom, then decrypt |

## Encryption Scheme

### Inline MIME (small emails)

MIME is stored in plaintext in the `content` field.

### Blossom MIME (large emails)

The MIME content is encrypted using AES-GCM.

```
RFC 2822 MIME message
    ↓
Generate random 256-bit key and 96-bit nonce
    ↓
AES-GCM Encrypt (key, nonce, mime_bytes)
    ↓
Encrypted MIME blob
    ↓
Upload to recipient Blossom servers
    ↓
Create kind 1301 with x tag + decryption key + nonce
```
