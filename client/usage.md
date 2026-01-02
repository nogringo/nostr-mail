---
label: Using the App
icon: inbox
order: 10
---

# Using the App

How to send and receive emails with the Nostr Mail client.

---

## Inbox

The inbox shows all received emails:

- Emails are sorted by date (newest first)
- Tap an email to view details
- Pull to refresh for new emails

### Real-Time Updates

New emails appear automatically as they arrive via Nostr relays.

---

## Composing Emails

### New Email

1. Tap the compose button (+ icon)
2. Fill in the recipient
3. Add a subject
4. Write your message
5. Tap "Send"

### Recipient Formats

| Format | Example | Description |
|--------|---------|-------------|
| NIP-05 | `alice@nostr.com` | Human-readable Nostr address |
| npub | `npub1abc...` | Nostr public key |
| Email | `alice@gmail.com` | Legacy email (needs bridge) |

---

## Sending to Legacy Email

To send to regular email addresses (Gmail, etc.):

1. You need a bridge account (e.g., `you@bridge.mail`)
2. Your "From" address determines which bridge to use
3. The bridge forwards your email via SMTP

```
From: you@bridge.mail
To: alice@gmail.com
       ↓
  Bridge receives via Nostr
       ↓
  Bridge sends via SMTP
       ↓
  alice@gmail.com receives
```

---

## Reading Emails

Tap any email in the inbox to view:

- **From**: Sender address and Nostr pubkey
- **To**: Recipient address
- **Subject**: Email subject
- **Date**: When the email was sent
- **Body**: Email content

### Actions

- **Reply**: Compose a reply
- **Delete**: Remove from local storage

---

## Profile

Access your profile to:

- View your public key (npub)
- View your NIP-05 address (if configured)
- Change settings
- Logout

---

## Settings

### Theme

Switch between light and dark mode.

### Relays

View connected relays and their status.

### Storage

- View cached emails count
- Clear local cache

---

## Tips

### Offline Access

Emails are cached locally. You can read previously synced emails offline.

### Performance

- The app syncs historical emails on first login
- New emails arrive in real-time via subscriptions
- Pull to refresh forces a re-sync

### Privacy

- All emails are end-to-end encrypted (NIP-59)
- Relays only see encrypted blobs
- Your private key never leaves your device

---

## Troubleshooting

### Emails Not Loading

1. Check internet connection
2. Pull to refresh
3. Check relay connectivity in settings

### Can't Send Email

1. Verify recipient format
2. For legacy email, ensure "From" has bridge domain
3. Check relay connectivity

### App Crashes

1. Clear app cache
2. Reinstall the app
3. Report issue on GitHub
