---
label: Configuration
icon: gear
order: 10
---

# Configuration

Complete configuration reference for all bridge components.

---

## Environment Variables

### Common Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `RELAYS` | Comma-separated relay URLs | Yes | `wss://relay.damus.io` |
| `PLUGIN_PATH` | Path to filter plugin executable | No | None |

---

## Inbound Service

### SMTP Mode

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `INBOUND_PRIVATE_KEY` | Nostr private key (hex) | Yes | - |
| `SMTP_PORT` | SMTP server port | No | `25` |
| `SMTP_HOST` | SMTP bind address | No | `0.0.0.0` |
| `RELAYS` | Relay URLs | Yes | - |
| `PLUGIN_PATH` | Filter plugin path | No | None |

### Mailgun Mode

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `INBOUND_PRIVATE_KEY` | Nostr private key (hex) | Yes | - |
| `MAILGUN_WEBHOOK_SECRET` | Webhook signature validation | Yes | - |
| `HTTP_PORT` | HTTP server port | No | `3001` |
| `RELAYS` | Relay URLs | Yes | - |
| `PLUGIN_PATH` | Filter plugin path | No | None |

---

## Outbound Service

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `BRIDGE_PRIVATE_KEY` | Nostr private key (hex) | Yes | - |
| `RELAYS` | Relay URLs | Yes | - |
| `OUTBOUND_PROVIDER` | `smtp` or `mailgun` | Yes | - |
| `FROM_DOMAIN` | Domain for From addresses | Yes | - |

### SMTP Provider

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `SMTP_HOST` | SMTP server hostname | Yes | - |
| `SMTP_PORT` | SMTP server port | No | `25` |
| `SMTP_SECURE` | Use TLS | No | `false` |
| `SMTP_USER` | SMTP username | No | None |
| `SMTP_PASS` | SMTP password | No | None |

### Mailgun Provider

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `MAILGUN_API_KEY` | Mailgun API key | Yes | - |
| `MAILGUN_DOMAIN` | Mailgun domain | Yes | - |

---

## NIP-05 Service

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `BRIDGE_PUBKEY` | Bridge's public key (hex) | Yes | - |
| `USER_STORE_PATH` | Path to users JSON file | No | `./data/users.json` |
| `HTTP_PORT` | HTTP server port | No | `3000` |

---

## Example Configurations

### Minimal Self-Hosted Setup

```bash
# .env
INBOUND_PRIVATE_KEY=<your-hex-key>
BRIDGE_PRIVATE_KEY=<your-hex-key>
RELAYS=wss://relay.damus.io,wss://nos.lol
OUTBOUND_PROVIDER=smtp
FROM_DOMAIN=mail.yourdomain.com
SMTP_HOST=localhost
SMTP_PORT=25
```

### Mailgun Setup

```bash
# .env
INBOUND_PRIVATE_KEY=<your-hex-key>
BRIDGE_PRIVATE_KEY=<your-hex-key>
MAILGUN_API_KEY=key-xxxxxxxxxx
MAILGUN_DOMAIN=mail.yourdomain.com
MAILGUN_WEBHOOK_SECRET=xxxxxxxxxx
RELAYS=wss://relay.damus.io,wss://nos.lol
OUTBOUND_PROVIDER=mailgun
FROM_DOMAIN=mail.yourdomain.com
HTTP_PORT=3001
```

### With Plugin Filtering

```bash
# .env
INBOUND_PRIVATE_KEY=<your-hex-key>
RELAYS=wss://relay.damus.io,wss://nos.lol
PLUGIN_PATH=/opt/nostr-mail/plugins/uid_ovh
```

---

## Docker Compose

```yaml
version: '3.8'

services:
  inbound:
    build: ./bridge-inbound/smtp
    ports:
      - "25:25"
    environment:
      - INBOUND_PRIVATE_KEY=${INBOUND_PRIVATE_KEY}
      - RELAYS=${RELAYS}
    restart: unless-stopped

  outbound:
    build: ./bridge-outbound
    environment:
      - BRIDGE_PRIVATE_KEY=${BRIDGE_PRIVATE_KEY}
      - RELAYS=${RELAYS}
      - OUTBOUND_PROVIDER=smtp
      - SMTP_HOST=postfix
      - FROM_DOMAIN=${FROM_DOMAIN}
    restart: unless-stopped

  nip05:
    build: ./nip05-service
    ports:
      - "3000:3000"
    environment:
      - BRIDGE_PUBKEY=${BRIDGE_PUBKEY}
    volumes:
      - ./data:/app/data
    restart: unless-stopped

  postfix:
    image: postfix:latest
    restart: unless-stopped
```

---

## DNS Configuration

For a fully functional bridge, configure these DNS records:

| Type | Name | Value |
|------|------|-------|
| A | `mail.yourdomain.com` | Your server IP |
| MX | `yourdomain.com` | `mail.yourdomain.com` |
| TXT | `_smtp.yourdomain.com` | NIP-05 discovery |

### SPF Record (optional but recommended)

```
v=spf1 mx ip4:YOUR_SERVER_IP -all
```

### DKIM (optional but recommended)

Configure DKIM signing in your MTA for better deliverability.
