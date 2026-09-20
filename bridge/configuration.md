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
| `PLUGIN_TIMEOUT_MS` | How long a plugin may take before it is given up on | No | `30000` |
| `SEND_DM_COPY` | Send a DM copy of inbound email. The user's `dm_copy` setting is what asks for it | No | `false` |

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
| `SMTP_REJECT_UNAUTHORIZED` | Verify the server certificate. Set to `false` only for a local MTA with a self-signed certificate | No | `true` |

### Mailgun Provider

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `MAILGUN_API_KEY` | Mailgun API key | Yes | - |
| `MAILGUN_DOMAIN` | Mailgun domain | Yes | - |
| `MAILGUN_REGION` | `us` or `eu` | No | `us` |

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

The repository ships a `docker-compose.yml` that runs the four services
together. Fill in `.env` and start it:

```bash
cp .env.example .env
docker compose up -d
```

```bash
# .env
INBOUND_PRIVATE_KEY=
BRIDGE_PRIVATE_KEY=
BRIDGE_PUBKEY=
FROM_DOMAIN=mail.example.com
# PLUGIN_PATH=/app/plugins/whitelist.js
```

| Service | Role |
|---------|------|
| `inbound-smtp` | SMTP on port 25, publishes to Nostr |
| `bridge` | Nostr to SMTP, sending through the bundled Postfix |
| `nip05-service` | `/.well-known/nostr.json` on port 3000 |
| `postfix` | Outbound MTA, with DKIM signing |

DKIM keys live in `./dkim-keys`, the NIP-05 user store in `./nip05-data`.
Back up both.

---

## DNS Configuration

Mail that is not authenticated goes to spam, so treat these as required rather
than optional.

| Type | Name | Value |
|------|------|-------|
| A | `mail.yourdomain.com` | Your server IP |
| MX | `yourdomain.com` | `mail.yourdomain.com` |
| TXT | `yourdomain.com` | `v=spf1 mx -all` |
| TXT | `<selector>._domainkey.yourdomain.com` | Your DKIM public key |
| TXT | `_dmarc.yourdomain.com` | `v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com` |

Also set the reverse DNS of your server IP to `mail.yourdomain.com`. Many
receivers reject mail from an address that does not resolve back.

### NIP-05 discovery

The bridge is discovered over HTTPS, not DNS: serve its pubkey under `_smtp` in
`https://yourdomain.com/.well-known/nostr.json`. That is what `nip05-service`
does.
