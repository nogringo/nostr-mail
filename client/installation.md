---
label: Installation
icon: download
order: 30
---

# Installation

---

## Install the app

Nothing to build: use [app.nostrmail.org](https://app.nostrmail.org), install
from [ZapStore](https://zapstore.dev/apps/app.nostrmail.client), or download an
Android, Linux or macOS build from the
[releases page](https://github.com/nogringo/nostr-mail-client/releases/latest).

The rest of this page is for building from source.

---

## Prerequisites

- [Flutter](https://flutter.dev/docs/get-started/install) with Dart SDK 3.12.2
  or newer

```bash
flutter doctor
```

---

## Clone

```bash
git clone https://github.com/nogringo/nostr-mail-client
cd nostr-mail-client
flutter pub get
```

The repository is a Flutter workspace. `flutter pub get` at the root resolves
`packages/nmail_core` and both app wrappers together.

| Path | What it is |
|------|------------|
| `packages/nmail_core` | The product code, shared by every build |
| `apps/nmail_standard` | The standard app, may depend on Firebase |
| `apps/nmail_foss` | The FOSS app, no Google dependency, UnifiedPush for notifications |

The FOSS wrapper carries the ZapStore build as an Android flavor, so the two
distributions share one app and one feature set.

---

## Run

Run from an app wrapper, not from the root:

```bash
cd apps/nmail_foss
flutter run -d chrome    # web
flutter run -d linux     # Linux desktop
flutter run -d macos     # macOS desktop
flutter run -d <device>  # Android
```

Use `apps/nmail_standard` for the build that includes Firebase messaging.

---

## Build

+++ Android
```bash
cd apps/nmail_foss
flutter build apk --release
```
+++ Web
```bash
cd apps/nmail_foss
flutter build web --release
```
+++ Linux
```bash
cd apps/nmail_foss
flutter build linux --release
```
+++ macOS
```bash
cd apps/nmail_foss
flutter build macos --release
```
+++

!!!warning Web builds
The mailbox is a SQLite database running in the browser. `sqlite3.wasm` and
`drift_worker.js` have to be present in `web/`, matching the resolved `sqlite3`
and `drift` versions. See [Local Storage](/sdk/storage/).
!!!

---

## Troubleshooting

```bash
flutter clean && flutter pub get
```

If a dependency fails to resolve, check that your Dart SDK matches the
`environment` constraint in `pubspec.yaml`. The client tracks the SDK closely
and tends to need a recent Flutter.
