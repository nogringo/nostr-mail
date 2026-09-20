---
label: Delivery Status
icon: report
order: 25
---

# Delivery Status Notifications

SMTP tells a sender what happened to their message with a bounce. A bridge does
the same with a kind 7679 event, without telling the relays who it is talking
to.

---

## The event

```json
{
  "kind": 7679,
  "pubkey": "<bridge_permanent_pubkey>",
  "tags": [
    ["r", "<email_id>"],
    ["ephemeral-pubkey", "<temp_pubkey>"]
  ],
  "content": "<nip44_encrypted_content>"
}
```

| Tag | Required | Description |
|-----|----------|-------------|
| `r` | Yes | The `email-id` of the original kind 1301 event |
| `ephemeral-pubkey` | Yes | Temporary public key, hex, used to decrypt the content |

---

## Why it is shaped this way

| Threat | Protection |
|--------|------------|
| Relays see who sent the original | No `p` tag, so the event names nobody |
| The bridge reads its own archive | The ephemeral private key is destroyed, leaving ciphertext |
| Delivery failures end up in logs | The content is encrypted, so the logs say nothing |
| A forged notification | The bridge's permanent key signs the event |

The sender is found through the `r` tag: only someone who knows the `email-id`
of an email they sent can ask for its notifications.

---

## Content

NIP-44 encrypted JSON:

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

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `delivered`, `failed`, `delayed` or `queued` |
| `recipient` | string | The recipient's email address |
| `smtp_code` | number | The SMTP response code |
| `smtp_response` | string | The full SMTP response |
| `attempted_at` | number | When delivery was attempted |
| `delivered_at` | number | Present on success |
| `failed_at` | number | Present on failure |
| `details` | string | Human-readable explanation |

| Status | SMTP codes | Meaning |
|--------|-----------|---------|
| `delivered` | 250, 2xx | Accepted by the recipient's server |
| `failed` | 5xx | Permanent failure: no such mailbox, refused, full |
| `delayed` | 4xx | Temporary: greylisting, rate limiting |
| `queued` | none | Accepted by the bridge, delivery still to come |

---

## Sending one, as a bridge

1. Generate an ephemeral keypair.
2. Encrypt the JSON with NIP-44 using the ephemeral private key and the sender's
   pubkey.
3. Build the kind 7679 event with the `r` and `ephemeral-pubkey` tags, signed by
   the bridge's permanent key.
4. Destroy the ephemeral private key. Never write it to disk.
5. Publish.

---

## Reading one, as a sender

```json
{
  "authors": ["<bridge_pubkey>"],
  "kinds": [7679],
  "#r": ["<email_id>"]
}
```

Take `temp_pubkey` from the `ephemeral-pubkey` tag, decrypt the content with
NIP-44 using your private key and that pubkey, and read the JSON.

This is why the `email-id` tag matters on the way out: without it, there is
nothing for a notification to point at.
