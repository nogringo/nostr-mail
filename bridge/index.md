---
label: Bridge
icon: server
order: 70
expanded: true
---

# Nostr Mail Bridge

A bridge is what lets a Nostr Mail user and an ordinary email user write to each
other. It speaks SMTP on one side, Nostr on the other, and converts nothing but
the transport: the RFC 2822 message crosses unchanged.

Reference implementation:
[nogringo/nostr-mail-bridge](https://github.com/nogringo/nostr-mail-bridge).

---

## Architecture

```
                         INBOUND (Email → Nostr)
┌─────────────┐      ┌─────────────────┐      ┌────────┐      ┌──────┐
│ Legacy Email│ ───► │ bridge-inbound  │ ───► │ Relays │ ───► │ User │
└─────────────┘      │ smtp or mailgun │      └────────┘      └──────┘
                     └─────────────────┘

                         OUTBOUND (Nostr → Email)
┌──────┐      ┌────────┐      ┌─────────────────┐      ┌─────────────┐
│ User │ ───► │ Relays │ ───► │ bridge-outbound │ ───► │ Legacy Email│
└──────┘      └────────┘      └─────────────────┘      └─────────────┘

                         DISCOVERY
┌────────┐      GET /.well-known/nostr.json      ┌───────────────┐
│ Client │ ──────────────────────────────────►   │ nip05-service │
└────────┘                                       └───────────────┘
```

---

## Components

| Component | Description | Port |
|-----------|-------------|------|
| `bridge-inbound` | Receives email and publishes it to Nostr | 25 / 3001 |
| `bridge-outbound` | Listens to Nostr and sends email | - |
| `nip05-service` | NIP-05 discovery, `/.well-known/nostr.json` | 3000 |
| `bridge-plugins` | Filtering plugins, one executable per policy | - |

---

## Quick Navigation

||| [:icon-download: Inbound](inbound.md)
Email to Nostr: SMTP server or Mailgun webhooks
||| [:icon-upload: Outbound](outbound.md)
Nostr to email: subscribe, unwrap, send
||| [:icon-report: Delivery Status](dsn.md)
Kind 7679 notifications, and why they name nobody
||| [:icon-plug: Plugins](plugins.md)
Accept, reject or silently drop
||| [:icon-gear: Configuration](configuration.md)
Environment variables and DNS
|||

---

## What a bridge has to honour

### The routing tags

An outbound kind 1301 rumor carries its envelope in its tags, not in the
headers. Read them from the tags, not from `To:`:

| Tag | Use |
|-----|-----|
| `email-id` | Quote it in every delivery status notification |
| `mail-from` | The SMTP envelope sender |
| `rcpt-to` | The SMTP envelope recipient, once per recipient |

Inbound, the bridge sets `mail-from` on the rumor it builds, so the recipient's
client can tell a bridged email from a native one. There is no `rcpt-to`
inbound: the recipient is the `p` tag of the gift wrap.

### The user's public settings

A user's kind 30078 event at `d = nostr-mail/settings` is public because
bridges read it:

| Field | Meaning for the bridge |
|-------|------------------------|
| `dm_copy` | Send a DM copy of incoming email |
| `prefer_nostr` | This key's NIP-05 addresses want mail over Nostr, not SMTP |

### Delivery status

Report every send with a kind 7679 event. See [Delivery Status](dsn.md).

---

## Quick Start with Docker

```bash
git clone https://github.com/nogringo/nostr-mail-bridge
cd nostr-mail-bridge
cp .env.example .env
# Fill in the keys and your domain
docker compose up -d
```

The compose file runs the inbound SMTP service, the outbound bridge, the NIP-05
service and a Postfix with DKIM signing.

---

## Another way in

Receiving mail does not have to mean running an SMTP server. A Haraka or
Mailgun front end can hand raw MIME to a webhook that does the Nostr side:

- [haraka-webhook](https://github.com/nogringo/haraka-webhook): a Haraka
  receiver that spools inbound mail and forwards it to a Mailgun-style MIME
  webhook.
- [nostr-mail-inbound-webhook](https://github.com/nogringo/nostr-mail-inbound-webhook):
  a Dart webhook that resolves recipients, gift wraps the MIME and publishes it,
  Blossom included for large messages.
- [nmail-api](https://github.com/nogringo/nmail-api): NIP-05 identity and
  inbound mail policy, which is also where the
  [alias protocol](https://github.com/nogringo/protocols/blob/main/manage-nip05.md)
  is served.

---

## Self-Hosted Setup

```
┌──────────────────────────────────────────────────┐
│                  Your Server                     │
│                                                  │
│  ┌──────────────┐  ┌────────┐  ┌──────────────┐  │
│  │bridge-inbound│  │outbound│  │ nip05-service│  │
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

Required DNS:

- MX record pointing to your server
- A or AAAA record for the mail domain
- SPF, DKIM and DMARC, or your mail will be filed as spam

Publish the bridge pubkey under `_smtp` in your `/.well-known/nostr.json`, which
is how a client discovers it.
