---
label: Client
icon: device-mobile
order: 50
expanded: true
---

# Nmail

The reference Nostr Mail client, built with Flutter.

[!button icon="mail" text="Open Nmail" target="blank"](https://app.nostrmail.org)
[!button icon="device-mobile" text="Get it on ZapStore" target="blank" variant="secondary"](https://zapstore.dev/apps/app.nostrmail.client)

Source: [nogringo/nostr-mail-client](https://github.com/nogringo/nostr-mail-client).

---

## Where to get it

| Platform | Where |
|----------|-------|
| Web | [app.nostrmail.org](https://app.nostrmail.org) |
| Android | [ZapStore](https://zapstore.dev/apps/app.nostrmail.client), or an APK from [Releases](https://github.com/nogringo/nostr-mail-client/releases/latest) |
| Linux | `.AppImage` or `.deb` from [Releases](https://github.com/nogringo/nostr-mail-client/releases/latest) |
| macOS | `.dmg` from [Releases](https://github.com/nogringo/nostr-mail-client/releases/latest) |

Android ships in two flavors: the standard build, which may use Firebase for
push, and a FOSS build with no Google dependency, which uses UnifiedPush
instead. ZapStore distributes the FOSS build.

---

## Features

- Inbox, Sent, Archive and Trash, with stars, read state and custom folders
- Rich text composing, attachments of any size, drag and drop
- Send to Nostr recipients and to ordinary email addresses in the same message,
  choosing the transport per recipient
- Scheduled sending
- Contacts shared with any client that speaks the same address book
- Multiple accounts, switched without logging out
- Offline reading, with sends and uploads queued until you are back
- Push notifications
- Light and dark themes, following the system

---

## Privacy in the app

- Emails are gift wrapped end to end. Relays see a wrap addressed to a pubkey.
- Folders, read state and stars are wrapped too, so the metadata does not leak.
- Photos and videos lose their location, capture date and device details before
  they leave the app, at full quality.
- Your key never leaves the device, and does not have to be on it at all: a
  signer app, a browser extension or a remote bunker can hold it instead.

---

## Technology

| Technology | Purpose |
|------------|---------|
| Flutter | Android, Linux, macOS and web from one codebase |
| GetX, go_router | State management and routing |
| [ndk](https://pub.dev/packages/ndk) | Nostr protocol, relays, signers |
| [nostr_mail](https://pub.dev/packages/nostr_mail) | Email over Nostr |
| drift, SQLite | Local mailbox with full-text search |
| Blossom | Large emails and attachments |

The repository is a Flutter workspace: `packages/nmail_core` holds the product
code, `apps/nmail_standard` and `apps/nmail_foss` are the two thin app
wrappers.
