# Nostr Mail Bridges

## Overview

Bridges act as gateways between Nostr and legacy email infrastructure, enabling Nostr users to send/receive emails to/from recipients who are not on Nostr.

## Sending Emails to Bridges

### Kind 1301 Event Structure

Emails destined for non-Nostr recipients are sent with additional routing tags:

```json
{
  "kind": 1301,
  "pubkey": "<sender_pubkey>",
  "tags": [
    ["email-id", "<unique_id>"],
    ["mail-from", "<sender_email>"],
    ["rcpt-to", "<recipient_email>"]
  ],
  "content": "<RFC 2822 email>"
}
```

### Delivery Flow

1. Sender creates kind 1301 event with `email-id`, `mail-from` and `rcpt-to` tags
2. Sender gift wraps the event (NIP-59)
3. Sender sends to bridge's DMs relays
4. Bridge receives and unwraps the gift wrap
5. Bridge extracts `email-id` and SMTP routing information from tags
6. Bridge delivers email via SMTP
7. Bridge sends DSN (Delivery Status Notification) back to sender

### Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `email-id` | Recommended | Unique random identifier for this email (used in DSN references) |
| `mail-from` | Yes | Sender's email address (used in SMTP MAIL FROM) |
| `rcpt-to` | Yes | Recipient's email address (used in SMTP RCPT TO) |
| Multiple `rcpt-to` tags MAY be used for CC/BCC recipients |

### Example

```json
{
  "kind": 1301,
  "pubkey": "alice_pubkey",
  "tags": [
    ["email-id", "550e8400-e29b-41d4-a716-446655440000"],
    ["mail-from", "npub1alice...@bridge.com"],
    ["rcpt-to", "bob@example.com"]
  ],
  "content": "From: npub1alice...@bridge.com\nTo: bob@example.com\nSubject: Hello\nDate: Sat, 28 Dec 2024 12:00:00 +0000\n\nHey Bob, how are you?"
}
```

## Delivery Status Notifications (DSN)

Bridges send delivery status notifications to inform senders about the outcome of email delivery attempts.

### Kind 1302: Delivery Status Notification

The DSN event allows senders to track whether their emails were successfully delivered and, if not, understand why delivery failed.

### Privacy Design

To protect user privacy, DSNs implement forward secrecy at the bridge level:

1. **No p tag**: Recipient metadata is not exposed on public relays
2. **Ephemeral encryption**: Content is encrypted with a temporary key that the bridge immediately destroys
3. **Permanent signature**: Event is signed with bridge's permanent key for authenticity

This means:
- Public relays cannot see who the original sender was (no p tag)
- Bridge cannot decrypt its own archived DSN events (temporary key destroyed)
- DSN authenticity is still verifiable via bridge's permanent signature

### Event Structure

```json
{
  "kind": 1302,
  "pubkey": "<bridge_permanent_pubkey>",
  "tags": [
    ["r", "<email_id>"],
    ["ephemeral-pubkey", "<temp_pubkey>"]
  ],
  "content": "<nip44_encrypted_content>"
}
```

### Tags

| Tag | Required | Description |
|-----|----------|-------------|
| `r` | Yes | Value from the `email-id` tag of the original kind 1301 email event |
| `ephemeral-pubkey` | Yes | Temporary public key (hex) for content decryption |

### Status Values

| Status | SMTP Codes | Description |
|--------|-----------|-------------|
| `delivered` | 250, 2xx | Email successfully delivered to recipient's SMTP server |
| `failed` | 5xx | Permanent delivery failure (invalid recipient, mailbox full, etc.) |
| `delayed` | 4xx | Temporary delay (greylisting, rate limiting, etc.) |
| `queued` | - | Email queued for delivery (bridge busy, retry scheduled) |

### Content (Encrypted)

The content field contains JSON data encrypted with NIP-44 using a temporary ephemeral key.

```json
{
  "status": "delivered",
  "recipient": "bob@example.com",
  "smtp_code": 250,
  "smtp_response": "2.1.5 OK",
  "attempted_at": 1735401600,
  "delivered_at": 1735401605,
  "details": "Email successfully delivered to recipient's SMTP server"
}
```

#### Content Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Delivery status (delivered, failed, delayed, queued) |
| `recipient` | string | The recipient's email address |
| `smtp_code` | number | The SMTP response code |
| `smtp_response` | string | The full SMTP response message |
| `attempted_at` | number | Unix timestamp when delivery was attempted |
| `delivered_at` | number (optional) | Unix timestamp when delivery was successful |
| `failed_at` | number (optional) | Unix timestamp when delivery failed |
| `details` | string | Human-readable explanation of the delivery outcome |

### Example: Successful Delivery

```json
{
  "kind": 1302,
  "pubkey": "bridge_permanent_pubkey",
  "tags": [
    ["r", "550e8400-e29b-41d4-a716-446655440000"],
    ["ephemeral-pubkey", "a7b3c9d2..."]
  ],
  "content": "<nip44_encrypted: {\"status\":\"delivered\",\"recipient\":\"bob@example.com\",\"smtp_code\":250,\"smtp_response\":\"2.1.5 OK\",\"attempted_at\":1735401600,\"delivered_at\":1735401605,\"details\":\"Email successfully delivered\"}>"
}
```

### Example: Failed Delivery

```json
{
  "kind": 1302,
  "pubkey": "bridge_permanent_pubkey",
  "tags": [
    ["r", "550e8400-e29b-41d4-a716-446655440000"],
    ["ephemeral-pubkey", "e8d4f1a6..."]
  ],
  "content": "<nip44_encrypted: {\"status\":\"failed\",\"recipient\":\"bob@example.com\",\"smtp_code\":550,\"smtp_response\":\"5.1.1 The email account that you tried to reach does not exist\",\"attempted_at\":1735401600,\"failed_at\":1735401600,\"details\":\"Recipient address rejected\"}>"
}
```

## Encryption Mechanism

### Bridge Side (Sending DSN)

1. Generate ephemeral keypair: `temp_privkey` + `temp_pubkey`
2. Encrypt content with NIP-44 using `(temp_privkey, sender_pubkey)`
3. Create kind 1302 event with:
   - `pubkey = bridge_permanent_pubkey`
   - Sign with `bridge_permanent_privkey`
   - Tag `["r", email_id]` where `email_id` is from the original kind 1301's `email-id` tag
   - Tag `["ephemeral-pubkey", temp_pubkey]`
   - Encrypted content from step 2
4. **Immediately destroy** `temp_privkey` (delete from memory, never write to disk)
5. Publish event to relays

### Sender Side (Receiving DSN)

1. Listen for events matching filter:
   ```json
   {
     "authors": ["<bridge_pubkey>"],
     "kinds": [1302],
     "#r": ["<email_id>"]
   }
   ```
   where `<email_id>` is the value from the kind 1301's `email-id` tag
2. Extract `temp_pubkey` from `ephemeral-pubkey` tag
3. Decrypt content with NIP-44 using `(sender_privkey, temp_pubkey)`
4. Parse decrypted JSON content

### Security Properties

| Threat | Protection |
|--------|-----------|
| Public relays see sender metadata | ✅ No `p` tag → anonymous |
| Bridge reads archived DSNs | ✅ `temp_privkey` destroyed → ciphertext only |
| Bridge logs delivery failures | ✅ Encrypted content → unreadable logs |
| DSN authenticity | ✅ Signed with permanent bridge key |
| Third-party reads DSNs | ✅ NIP-44 encrypted content |

## Retry Logic

### Bridges SHOULD retry failed deliveries according to SMTP standards:

- **4xx errors**: Retry with exponential backoff (typically 1m, 5m, 15m, 1h, 4h, 16h)
- **5xx errors**: Do not retry (permanent failure, send DSN with `status: "failed"`)

### Bridge MUST send DSN when:

1. **Success**: Email delivered → DSN with `status: "delivered"`
2. **Permanent failure**: 5xx error after all retries exhausted → DSN with `status: "failed"`
3. **Temporary delay**: 4xx error or queued → DSN with `status: "delayed"` or `status: "queued"`

### Bridge MAY send intermediate DSNs:

For long delays (e.g., greylisting), bridge MAY send intermediate DSNs with status `delayed` to keep sender informed before final delivery or failure.
