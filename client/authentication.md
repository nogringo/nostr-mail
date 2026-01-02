---
label: Authentication
icon: key
order: 20
---

# Authentication

Key management and login in the Nostr Mail client.

---

## How It Works

The client uses Nostr keys for authentication:

1. **Private Key (nsec/hex)**: Your identity - never shared
2. **Public Key (npub/hex)**: Your address - shared publicly

Keys are stored securely using Flutter Secure Storage.

---

## Login Options

### Import Existing Key

If you already have a Nostr identity:

1. Open the app
2. Tap "Import Key"
3. Enter your private key (nsec or hex format)
4. Tap "Login"

### Generate New Key

To create a new identity:

1. Open the app
2. Tap "Generate New Key"
3. **Important**: Backup your key immediately!
4. Continue to inbox

---

## Key Formats

The client accepts multiple formats:

| Format | Example | Description |
|--------|---------|-------------|
| nsec | `nsec1abc...` | Bech32 encoded private key |
| hex | `0123456...` | 64 character hex string |

---

## Key Storage

Keys are stored using platform-specific secure storage:

| Platform | Storage Method |
|----------|---------------|
| iOS | Keychain |
| Android | Encrypted SharedPreferences |
| macOS | Keychain |
| Windows | Windows Credential Manager |
| Linux | libsecret |
| Web | Encrypted localStorage |

---

## Security Best Practices

!!!danger Backup Your Key
Your private key is the only way to access your account. If lost, your identity cannot be recovered.
!!!

### Do

- Back up your key in a secure location
- Use a password manager
- Consider a hardware backup

### Don't

- Share your private key
- Store it in plain text
- Take screenshots of it

---

## Logout

To log out:

1. Go to Profile
2. Tap "Logout"
3. Confirm

!!!warning
Logging out removes the key from the device. Make sure you have a backup!
!!!

---

## NIP-07 Extension (Web)

On web, you can use browser extensions like:

- [nos2x](https://github.com/nicholasmfraser/nos2x)
- [Alby](https://getalby.com/)

The client will detect and use the extension for signing.

---

## Multiple Accounts

Currently, the client supports one account at a time. To switch accounts:

1. Logout
2. Login with different key
