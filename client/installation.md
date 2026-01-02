---
label: Installation
icon: download
order: 30
---

# Installation

Build and run the Nostr Mail client on your platform.

---

## Prerequisites

- [Flutter SDK](https://flutter.dev/docs/get-started/install) 3.10.4+
- [Dart SDK](https://dart.dev/get-dart) 3.10.4+

Verify your installation:

```bash
flutter doctor
```

---

## Clone & Setup

```bash
# Clone the repository
git clone https://github.com/nogringo/nostr-mail-client
cd nostr-mail-client

# Install dependencies
flutter pub get
```

---

## Run the App

### Development

```bash
# Default platform
flutter run

# Specific platform
flutter run -d chrome       # Web
flutter run -d macos        # macOS
flutter run -d windows      # Windows
flutter run -d linux        # Linux
```

### Release Build

+++ Android
```bash
flutter build apk --release
# Output: build/app/outputs/flutter-apk/app-release.apk
```
+++ iOS
```bash
flutter build ios --release
# Then archive with Xcode
```
+++ Web
```bash
flutter build web --release
# Output: build/web/
```
+++ macOS
```bash
flutter build macos --release
# Output: build/macos/Build/Products/Release/
```
+++ Windows
```bash
flutter build windows --release
# Output: build/windows/runner/Release/
```
+++ Linux
```bash
flutter build linux --release
# Output: build/linux/x64/release/bundle/
```
+++

---

## Platform-Specific Setup

### Android

No additional setup required.

### iOS

1. Open `ios/Runner.xcworkspace` in Xcode
2. Set your development team
3. Configure signing

### Web

Add to `web/index.html` if needed for CORS:

```html
<script>
  // CORS proxy configuration if needed
</script>
```

### Desktop (macOS/Windows/Linux)

Enable desktop support:

```bash
flutter config --enable-macos-desktop
flutter config --enable-windows-desktop
flutter config --enable-linux-desktop
```

---

## Configuration

### Relays

The default relays are configured in the app. To customize:

1. Open `lib/services/ndk_service.dart`
2. Modify the `bootstrapRelays` list

```dart
final ndk = Ndk(NdkConfig(
  bootstrapRelays: [
    'wss://relay.damus.io',
    'wss://nos.lol',
    // Add your relays here
  ],
));
```

---

## Troubleshooting

### Dependency Issues

```bash
flutter clean
flutter pub get
```

### Build Errors

```bash
# Update Flutter
flutter upgrade

# Regenerate platform files
flutter create --platforms=android,ios,web .
```

### Rust Issues (NDK)

The NDK uses Rust for signature verification. If you encounter issues:

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add targets
rustup target add aarch64-linux-android
rustup target add armv7-linux-androideabi
rustup target add x86_64-linux-android
rustup target add i686-linux-android
```
