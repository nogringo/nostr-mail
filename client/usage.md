---
label: Using the App
icon: inbox
order: 10
---

# Using Nmail

---

## Your address

Out of the box you are `<npub>@nostr`, which works between Nostr Mail users and
nowhere else. Copy it from the drawer or the account menu.

To be reachable from ordinary email, claim a readable address on a NIP-05
domain, for example `alice@example.com`. That address is what a Gmail user
types, and a bridge turns their message into mail for you.

---

## Reading

The inbox lists what arrived, newest first, with the sender, the subject, a
preview of the body as it reads and the attachments. Unread, starred and
attachment states show on the row.

Open an email to read it, reply, forward, star it, move it to another folder, or
save an attachment. Images and PDFs open in a viewer without leaving the app.

Everything is cached locally, so the mailbox works offline and search is
instant. Pull to refresh when you want to go to the relays now.

---

## Writing

Compose, then type a recipient: a Nostr address, an npub, or an ordinary email
address. Rich text, attachments and drag and drop all work.

### Choosing the transport per recipient

Tap a recipient while writing to decide how they receive the email: over Nostr,
or through SMTP. You can also move them between To, Cc and Bcc, or edit the
address.

The app picks a sensible default. An address that resolves on Nostr is delivered
over Nostr, unless its owner asks for SMTP. A reply goes back the way the
message arrived, because an email that came in over SMTP has a sender who will
never see a Nostr reply.

### Scheduling

Pick a send time and the email is handed to a scheduling service, which
publishes it at that moment. Scheduled emails have their own list, showing where
each one stands: awaiting confirmation, scheduled, sending, or overdue. Cancel
or edit one until it starts sending.

---

## Organising

| Action | Effect |
|--------|--------|
| Star | Keeps it in Starred |
| Archive | Out of the inbox, still searchable |
| Trash | Recoverable until you empty it |
| Folder | Any folder you name |
| Read / unread | Toggle at will |

These are private: each one is a signed label in its own gift wrap, so the
relays learn nothing from them. They follow you to your other devices.

---

## Contacts

The address book is shared with any client that speaks the same list, and is
private by default.

---

## Settings

- **Identities**: the From addresses you can send as, and the default one
- **Signature**: appended to what you write
- **Bridges**: the SMTP bridges you use
- **Relays**: where your mail arrives, at least one required
- **Notifications**: per account
- **Theme**: light, dark or system

Identities, signature and bridges are stored encrypted to yourself, so they sync
between devices without anyone else reading them.

---

## Troubleshooting

**No mail arrives.** Check the relay list. Mail arrives on the relays it names,
so an account whose relays changed may be looking in the wrong place.

**An email will not send.** Sends are queued and retried, so a failure usually
means the recipient could not be resolved. Check the address, and whether it is
meant to go over Nostr or SMTP.

**A signer keeps asking.** A remote signer is asked to decrypt each wrap. Keep
the signer reachable, and approve the session rather than each request where
your signer allows it.
