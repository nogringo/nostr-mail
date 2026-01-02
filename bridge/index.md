---
label: Bridge
icon: server
order: 70
expanded: true
---

# Nostr Mail Bridge

A bidirectional bridge between legacy email systems and the Nostr protocol.

---

## Architecture

```
                         INBOUND (Email → Nostr)
┌─────────────┐      ┌─────────────────┐      ┌────────┐      ┌──────┐
│ Legacy Email│ ───► │ inbound-smtp    │ ───► │ Relays │ ───► │ User │
└─────────────┘      │ or              │      └────────┘      └──────┘
                     │ inbound-mailgun │
                     └─────────────────┘

                         OUTBOUND (Nostr → Email)
┌──────┐      ┌────────┐      ┌────────┐      ┌─────────────┐
│ User │ ───► │ Relays │ ───► │ bridge │ ───► │ Legacy Email│
└──────┘      └────────┘      └────────┘      └─────────────┘

                         DISCOVERY
┌────────┐      GET /.well-known/nostr.json      ┌───────────────┐
│ Client │ ──────────────────────────────────►   │ nip05-service │
└────────┘                                       └───────────────┘
```

---

## Components

| Component | Description | Port |
|-----------|-------------|------|
| `bridge-inbound` | Receives emails and publishes to Nostr | 25 / 3001 |
| `bridge-outbound` | Listens to Nostr and sends emails | - |
| `nip05-service` | NIP-05 user discovery | 3000 |
| `bridge-plugins` | Email filtering plugins | - |

---

## Quick Navigation

||| [:icon-download: Inbound](inbound.md)
Email → Nostr: SMTP server or Mailgun webhooks
||| [:icon-upload: Outbound](outbound.md)
Nostr → Email: Subscribe to events and send via SMTP
||| [:icon-plug: Plugins](plugins.md)
Extensible plugin system for filtering
||| [:icon-gear: Configuration](configuration.md)
Environment variables and settings
|||

---

## Quick Start with Docker

```bash
git clone https://github.com/nogringo/nostr-mail-bridge
cd nostr-mail-bridge
cp .env.example .env
docker-compose up -d
```

---

## Services Overview

### Inbound Service

Handles incoming emails from legacy systems:

- **SMTP Mode**: Runs an SMTP server to receive emails directly
- **Mailgun Mode**: Receives webhooks from Mailgun

### Outbound Service

Handles outgoing emails to legacy systems:

- Subscribes to gift-wrapped events (kind 1059)
- Unwraps NIP-59 encrypted emails
- Sends via SMTP or Mailgun API

### NIP-05 Service

Provides user discovery via `.well-known/nostr.json`:

```http
GET /.well-known/nostr.json?name=alice

{
  "names": {
    "alice": "<pubkey_hex>"
  }
}
```

---

## Self-Hosted Setup

For a fully self-hosted setup without external services:

```
┌──────────────────────────────────────────────────┐
│                  Your Server                     │
│                                                  │
│  ┌──────────────┐  ┌────────┐  ┌──────────────┐  │
│  │ inbound-smtp │  │ bridge │  │ nip05-service│  │
│  │    :25       │  │        │  │    :3000     │  │
│  └──────────────┘  └────────┘  └──────────────┘  │
│         │              │              │          │
│         └──────────────┼──────────────┘          │
│                        │                         │
│                   ┌────────┐                     │
│                   │ Postfix│ (or any MTA)        │
│                   └────────┘                     │
└──────────────────────────────────────────────────┘
```

Required DNS configuration:
- MX record pointing to your server
- A/AAAA record for your mail domain
