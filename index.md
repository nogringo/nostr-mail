---
label: Home
icon: home
order: 100
---

# Nostr Mail

Remove gatekeepers from email. Use Nostr as transport instead of SMTP between
users.

---

## What is Nostr Mail?

Nostr Mail carries ordinary RFC 2822 email over [Nostr](https://nostr.how).
Your address is derived from your keys, your messages are gift wrapped
end to end, and no provider stands between you and the person you write to. When
the other side is not on Nostr, a bridge speaks SMTP on your behalf.

!!!success Key Benefits
- **Decentralized**: no single point of failure or control
- **Censorship-resistant**: your messages go through Nostr relays
- **Privacy-focused**: end-to-end encrypted with NIP-59 gift wrapping
- **Interoperable**: works with legacy email via bridges
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
Dart and JavaScript libraries for building Nostr Mail applications.
[Learn more :icon-arrow-right:](/sdk/)
||| [!badge Client]
Nmail, the reference client for Android, Linux and the web.
[Learn more :icon-arrow-right:](/client/)
|||

---

## Try it

[!button icon="mail" text="Open Nmail" target="blank"](https://app.nostrmail.org)
[!button icon="globe" text="nostrmail.org" target="blank" variant="secondary"](https://nostrmail.org)

---

## Quick Links

- [:icon-book: Protocol Specification](/protocol/)
- [:icon-rocket: Getting Started](/getting-started/)
- [:icon-server: Bridge Setup](/bridge/)
- [:icon-package: SDKs](/sdk/)
- [:icon-device-mobile: Nmail Client](/client/)

---

## NIPs Used

| NIP | Description | Usage |
|-----|-------------|-------|
| [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) | Basic protocol | Event structure |
| [NIP-05](https://github.com/nostr-protocol/nips/blob/master/05.md) | User discovery | `user@domain` resolution |
| [NIP-09](https://github.com/nostr-protocol/nips/blob/master/09.md) | Event deletion | Deleting emails and labels |
| [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) | Private DM relays | Kind 10050 DM relays |
| [NIP-32](https://github.com/nostr-protocol/nips/blob/master/32.md) | Labeling | Folders, read state, stars |
| [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) | Encryption | Sealing and settings |
| [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) | Gift-wrapped events | End-to-end encryption |
| [NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md) | Relay list metadata | Relay discovery |
| [NIP-78](https://github.com/nostr-protocol/nips/blob/master/78.md) | Application data | Settings sync |

---

## License

This project is licensed under the MIT License.
