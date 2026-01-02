---
label: Client
icon: device-mobile
order: 50
expanded: true
---

# Flutter Client

A cross-platform email client for Nostr Mail built with Flutter.

[!button icon="globe" text="Try Online" target="blank"](https://nogringo.github.io/nostr-mail-client)

---

## Features

- Send and receive emails over Nostr
- Support for legacy email addresses via bridges
- Real-time inbox updates
- Secure key management with Flutter Secure Storage
- Light and dark themes
- Cross-platform (iOS, Android, Web, Desktop)

---

## Screenshots

Coming soon...

---

## Technology Stack

| Technology | Purpose |
|------------|---------|
| Flutter | Cross-platform UI |
| GetX | State management & routing |
| NDK | Nostr protocol |
| nostr_mail | Email over Nostr SDK |
| Sembast | Local storage |
| Toastification | Toast notifications |

---

## Architecture

The client follows the MVC pattern with GetX:

```
lib/
├── app/
│   ├── bindings/     # Dependency injection
│   └── routes/       # Navigation routes
├── controllers/      # Business logic
├── models/           # Data models
├── services/         # API & storage services
├── utils/            # Utilities
└── views/            # UI screens
    ├── auth/         # Login/Register
    ├── compose/      # Compose email
    ├── email/        # Email detail
    ├── inbox/        # Inbox list
    └── profile/      # User profile
```

---

## Quick Navigation

||| [:icon-download: Installation](installation.md)
Build and run the client
||| [:icon-key: Authentication](authentication.md)
Key management and login
||| [:icon-inbox: Using the App](usage.md)
How to use the email client
|||

---

## Requirements

- Flutter SDK 3.10.4 or higher
- Dart SDK 3.10.4 or higher

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/nogringo/nostr-mail-client
cd nostr-mail-client

# Install dependencies
flutter pub get

# Run the app
flutter run
```

---

## Dependencies

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_secure_storage: ^10.0.0
  get: ^4.7.3
  ndk: ^0.6.1-dev.4
  path_provider: ^2.1.5
  sembast: ^3.8.6
  toastification: ^3.0.3
  nostr_mail: ^1.1.0
  ndk_rust_verifier: ^0.4.2
  http: ^1.6.0
```
