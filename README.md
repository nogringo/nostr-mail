# Mail to Nostr

A bridge between legacy email and the Nostr protocol. Send and receive emails via Nostr using NIP-59 gift-wrapped messages.

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

## Services

| Service | Description | Port |
|---------|-------------|------|
| `bridge` | Listens to Nostr, sends email via SMTP or Mailgun | - |
| `inbound-smtp` | SMTP server, publishes to Nostr | 25 |
| `inbound-mailgun` | Mailgun webhook handler, publishes to Nostr | 3001 |
| `nip05-service` | Serves `/.well-known/nostr.json` for NIP-05 | 3000 |

## Quick Start

### 1. Build shared core (required for inbound services)

```bash
cd packages/inbound-core
npm install
npm run build
```

### 2. Choose your inbound service

**Option A: Self-hosted SMTP (no external service)**

```bash
cd packages/inbound-smtp
npm install
cp .env.example .env
# Edit .env with your keys
npm run build
sudo npm start  # Port 25 requires root
```

**Option B: Mailgun webhooks**

```bash
cd packages/inbound-mailgun
npm install
cp .env.example .env
# Edit .env with your keys
npm run build
npm start
```

### 3. Start the outbound bridge

```bash
cd bridge
npm install
cp .env.example .env
# Edit .env with your keys
npm run build
npm start
```

### 4. Start NIP-05 service

```bash
cd nip05-service
npm install
cp .env.example .env
# Edit .env with your keys
npm run build
npm start
```

## Configuration

### bridge/.env

```bash
BRIDGE_PRIVATE_KEY=<hex>
RELAYS=wss://relay.damus.io,wss://nos.lol

# Outbound provider: 'smtp' or 'mailgun'
OUTBOUND_PROVIDER=smtp
FROM_DOMAIN=mail.example.com

# SMTP settings (when OUTBOUND_PROVIDER=smtp)
SMTP_HOST=localhost
SMTP_PORT=25
SMTP_SECURE=false
SMTP_USER=
SMTP_PASS=

# Mailgun settings (when OUTBOUND_PROVIDER=mailgun)
MAILGUN_API_KEY=key-xxx
MAILGUN_DOMAIN=mail.example.com
```

### packages/inbound-smtp/.env

```bash
INBOUND_PRIVATE_KEY=<hex>
RELAYS=wss://relay.damus.io,wss://nos.lol
SMTP_PORT=25
SMTP_HOST=0.0.0.0
```

### packages/inbound-mailgun/.env

```bash
INBOUND_PRIVATE_KEY=<hex>
MAILGUN_WEBHOOK_SECRET=xxx
RELAYS=wss://relay.damus.io,wss://nos.lol
HTTP_PORT=3001
```

### nip05-service/.env

```bash
BRIDGE_PUBKEY=<hex>
USER_STORE_PATH=./data/users.json
HTTP_PORT=3000
```

## Email Addressing

### Address Formats Comparison

| Format | Example | Readable | Sovereign | RFC 2822 | Legacy Reply |
|--------|---------|----------|-----------|----------|--------------|
| `nip05@bridge.com` | `alice@bridge.com` | ✓ | ✗ | ✓ | ✓ |
| `npub@bridge.com` | `npub1...@bridge.com` | ✗ | ✓ | ✓ | ✓ |
| `npub@nostr` | `npub1...@nostr` | ✗ | ✓ | ✓ | ✗ |
| `npub` | `npub1...` | ✗ | ✓ | ✗ | ✗ |

### Format Details

#### `nip05@bridge.com`
- **Pros**: Human-readable, works everywhere, legacy clients can reply
- **Cons**: Not sovereign - depends on NIP-05 server to resolve your identity
- **Use case**: User-friendly email bridge service

#### `npub@bridge.com`
- **Pros**: Sovereign (npub derived from your keys), works with legacy clients
- **Cons**: Long and not human-readable, requires bridge infrastructure
- **Use case**: Sovereign identity with full legacy email compatibility

#### `npub@nostr`
- **Pros**: Sovereign, RFC 2822 compliant, email parsers work correctly
- **Cons**: `nostr` is not a real domain - legacy clients cannot reply
- **Use case**: Nostr-to-Nostr communication, forwarding to Gmail (read-only)

#### `npub` (no domain)
- **Pros**: Simplest format, fully sovereign
- **Cons**: Not RFC 2822 compliant, email parsers will fail to extract the address
- **Use case**: Not recommended - causes parsing issues

### NIP-05 Resolution

For NIP-05 names, the service queries `https://yourdomain.com/.well-known/nostr.json?name=alice` to resolve the pubkey.

## How It Works

### Inbound (Email → Nostr)

1. Email arrives at `alice@bridge.com`
2. Service extracts recipient pubkey (via NIP-05 lookup if needed)
3. Fetches recipient's DM relays (kind 10050)
4. Gift-wraps the email content (NIP-59)
5. Publishes to recipient's DM relays

### Outbound (Nostr → Email)

1. Bridge subscribes to gift-wrapped events (kind 1059) for its pubkey
2. Unwraps the event to extract email content
3. Sends via configured provider (SMTP or Mailgun)
4. From address: `<sender-npub>@yourdomain.com`

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

Configure:
- `inbound-smtp`: Receives incoming mail
- `bridge`: `OUTBOUND_PROVIDER=smtp` to send via local Postfix
- DNS: MX record pointing to your server
