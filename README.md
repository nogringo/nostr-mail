# Nostr Mail

Remove gatekeepers from email. Use Nostr as transport instead of SMTP between
users.

This repository holds the **documentation site**, published at
[nogringo.github.io/nostr-mail](https://nogringo.github.io/nostr-mail). The
project itself lives at [nostrmail.org](https://nostrmail.org).

## The project

| Piece | Repository | Where to get it |
|-------|------------|-----------------|
| Nmail, the reference client | [nostr-mail-client](https://github.com/nogringo/nostr-mail-client) | [app.nostrmail.org](https://app.nostrmail.org), [ZapStore](https://zapstore.dev/apps/app.nostrmail.client) |
| Dart SDK | [nostr-mail-dart](https://github.com/nogringo/nostr-mail-dart) | [pub.dev/packages/nostr_mail](https://pub.dev/packages/nostr_mail) |
| JavaScript SDK | [nostr-mail-js](https://github.com/nogringo/nostr-mail-js) | [npmjs.com/package/nostr-mail](https://www.npmjs.com/package/nostr-mail) |
| Bridge to legacy email | [nostr-mail-bridge](https://github.com/nogringo/nostr-mail-bridge) | self-hosted |
| Inbound webhook | [nostr-mail-inbound-webhook](https://github.com/nogringo/nostr-mail-inbound-webhook), [haraka-webhook](https://github.com/nogringo/haraka-webhook) | self-hosted |
| NIP-05 identity and inbound policy | [nmail-api](https://github.com/nogringo/nmail-api) | self-hosted |

## Specifications

The core, bridge and large MIME specs live in [`nostrhub/`](nostrhub/). The
labels, settings, contacts and alias specs live in
[nogringo/protocols](https://github.com/nogringo/protocols) and are published on
[openspecs](https://openspecs.uid.ovh/npub1kg4sdvz3l4fr99n2jdz2vdxe2mpacva87hkdetv76ywacsfq5leqquw5te).

| Spec | Source |
|------|--------|
| Nostr Mail Core | [`nostrhub/nostr-mail-core.md`](nostrhub/nostr-mail-core.md) |
| Bridges and delivery status | [`nostrhub/nostr-mail-bridge.md`](nostrhub/nostr-mail-bridge.md) |
| Large MIME on Blossom | [`nostrhub/nostr-mail-blossom.md`](nostrhub/nostr-mail-blossom.md) |
| Labels | [`nostrhub/nostr-mail-labels.md`](nostrhub/nostr-mail-labels.md) |
| Settings | [`nostrhub/nostr-mail-settings.md`](nostrhub/nostr-mail-settings.md) |
| Mail contacts | [protocols/mail-contacts.md](https://github.com/nogringo/protocols/blob/main/mail-contacts.md) |
| Alias protocol | [protocols/manage-nip05.md](https://github.com/nogringo/protocols/blob/main/manage-nip05.md) |

The `nostrhub/` files are the spec source and are excluded from the built site.
The site explains them in [`protocol/`](protocol/).

## Working on the docs

```bash
npm install
npm run docs:dev     # live preview
npm run docs:build   # build into .retype
npm run docs:serve   # serve the build
```

The site is built with [Retype](https://retype.com) and deployed to GitHub Pages
on every push to `main`, by
[`.github/workflows/docs.yml`](.github/workflows/docs.yml).

Layout:

| Path | Section |
|------|---------|
| `index.md` | Home |
| `getting-started/` | Where to start, per audience |
| `protocol/` | How the protocol works |
| `bridge/` | Running a bridge |
| `sdk/` | Dart and JavaScript SDKs |
| `client/` | Nmail |

## License

MIT
