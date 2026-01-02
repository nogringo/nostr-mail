---
label: Home
icon: home
order: 100
---

# Nostr Mail

Remove gatekeepers from email. Use Nostr as transport instead of SMTP between users.

---

## What is Nostr Mail?

Nostr Mail is a decentralized email system that uses the [Nostr protocol](https://nostr.com) as its transport layer. It allows you to send and receive emails without relying on traditional email providers.

!!!success Key Benefits
- **Decentralized**: No single point of failure or control
- **Censorship-resistant**: Your messages go through Nostr relays
- **Privacy-focused**: End-to-end encrypted with NIP-59 gift wrapping
- **Interoperable**: Works with legacy email via bridges
!!!

---

## Architecture Overview

```
                    NOSTR NATIVE
┌──────────┐                              ┌──────────┐
│  Alice   │ ────── Nostr Relays ──────►  │   Bob    │
│ (Client) │      Kind 1301 + NIP-59      │ (Client) │
└──────────┘                              └──────────┘

                    WITH LEGACY EMAIL
┌──────────┐      ┌────────┐      ┌──────────┐
│  Gmail   │ ───► │ Bridge │ ───► │   Bob    │
│   User   │ SMTP │        │Nostr │ (Client) │
└──────────┘      └────────┘      └──────────┘
```

---

## Components

||| [!badge Protocol]
The email-over-Nostr specification using Kind 1301 events.
[Learn more :icon-arrow-right:](/protocol/)
||| [!badge Bridge]
Bidirectional bridge between legacy email (SMTP) and Nostr.
[Learn more :icon-arrow-right:](/bridge/)
||| [!badge SDK]
Dart library for building Nostr Mail applications.
[Learn more :icon-arrow-right:](/sdk/)
||| [!badge Client]
Flutter application for sending and receiving emails.
[Learn more :icon-arrow-right:](/client/)
|||

---

## Quick Links

- [:icon-book: Protocol Specification](/protocol/)
- [:icon-rocket: Getting Started](/getting-started/)
- [:icon-server: Bridge Setup](/bridge/)
- [:icon-package: Dart SDK](/sdk/)
- [:icon-device-mobile: Flutter Client](/client/)

---

## NIPs Used

| NIP | Description | Usage |
|-----|-------------|-------|
| [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) | Basic protocol | Event structure |
| [NIP-05](https://github.com/nostr-protocol/nips/blob/master/05.md) | User discovery | `user@domain` resolution |
| [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) | Private DM relays | Kind 10050 DM relays |
| [NIP-51](https://github.com/nostr-protocol/nips/blob/master/51.md) | Lists | Whitelist management |
| [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) | Gift-wrapped events | End-to-end encryption |
| [NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md) | Relay list metadata | Relay discovery |

---

## License

This project is licensed under the MIT License.
