---
label: Authentication
icon: key
order: 20
---

# Accounts and Keys

Your Nostr key is your mailbox. There is no password and no account on a
server: whoever holds the key holds the mail.

---

## Creating an account

Create one in the app and it generates a key for you. The app shows a **sync
code**, which is that key in a form you can paste into another device to get
the same mailbox there.

!!!danger Back up the sync code
It is the only way back into your mailbox. Nobody can reset it, and nobody can
recover it for you. Keep it in a password manager, not in a screenshot.
!!!

---

## Signing in

| Method | Where it fits |
|--------|---------------|
| Sync code | The key itself. Simplest, and the key lives in the app |
| Signer app | Android signers such as Amber. The key stays in that app |
| Browser extension | On web, a NIP-07 extension signs without exposing the key |
| Remote bunker | NIP-46. The key stays on a machine you control |

The sync code is what the login screen asks for. The other three are behind
**More options**.

A signer keeps your key out of the app entirely: Nmail asks it to sign and to
decrypt, and it answers. That is the setup to use if the same key also carries
your social identity.

---

## Several accounts

Add as many as you like and switch between them from the account menu. Each one
keeps its own mailbox, its own settings and its own local data. Removing an
account clears what is on the device and leaves the key untouched.

---

## Relays

An account needs a relay list. Nmail asks for one before it lets you in, and
keeps at least one relay in it. Those relays are where your mail arrives, so an
account with none has no mailbox.

---

## Deleting an account

Deleting asks the relays to forget you: a signed NIP-62 vanish request, aimed at
every relay this account could have reached rather than just your own. Relays
that honour it drop your events, the gift wraps addressed to you included, since
a wrap signed by a throwaway key is recognised by its `p` tag. The local data
goes with it. The key itself stays yours, and can be used again later.
