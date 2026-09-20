# Nostr Mail Labels

This document defines the protocol for managing email metadata (folders, read state, stars, custom tags) in Nostr Mail using NIP-32 labels.

## Overview

Email metadata is managed through NIP-32 label events (kind 1985). Each label is a separate event, allowing granular control and easy synchronization across clients.

## Namespace

All Nostr Mail labels use the namespace: `mail`

## Label Format

### Adding a Label

To add a label to an email, sign a kind 1985 event and put it, without a seal, in a NIP-59 gift wrap (kind 1059) addressed to the user:

```json
{
  "kind": 1985,
  "pubkey": "<user_pubkey>",
  "tags": [
    ["L", "mail"],
    ["l", "<label>", "mail"],
    ["e", "<gift_wrap_event_id>", "", "labelled"]
  ],
  "content": ""
}
```

### Removing a Label

To remove a label, publish a NIP-09 deletion request (kind 5) targeting the gift wrap of the label event:

```json
{
  "kind": 5,
  "pubkey": "<user_pubkey>",
  "tags": [
    ["e", "<label_gift_wrap_id>"],
    ["k", "1059"]
  ],
  "content": ""
}
```

## Standard Labels

### Folders

Emails without a folder label are considered to be in the inbox (default state).

| Label | Description |
|-------|-------------|
| `folder:trash` | Email is in the trash |
| `folder:archive` | Email is archived |
| `folder:spam` | Email is marked as spam |
| `folder:<custom>` | Custom folder (user-defined) |

### Read State

Emails without a read state label are considered unread (default state).

| Label | Description |
|-------|-------------|
| `state:read` | Email has been read |

### Flags

Emails without flag labels have no special flags (default state).

| Label | Description |
|-------|-------------|
| `flag:starred` | Email is starred/favorited |
| `flag:important` | Email is marked as important |

### Custom Tags

Users can create custom tags for organization:

| Label | Description |
|-------|-------------|
| `tag:<name>` | Custom user-defined tag |

## Examples

### Move Email to Trash

```json
{
  "kind": 1985,
  "pubkey": "abc123...",
  "created_at": 1234567890,
  "tags": [
    ["L", "mail"],
    ["l", "folder:trash", "mail"],
    ["e", "def456...", "", "labelled"]
  ],
  "content": ""
}
```

### Mark Email as Read and Starred

Two separate events:

**Read event:**
```json
{
  "kind": 1985,
  "tags": [
    ["L", "mail"],
    ["l", "state:read", "mail"],
    ["e", "def456...", "", "labelled"]
  ],
  "content": ""
}
```

**Starred event:**
```json
{
  "kind": 1985,
  "tags": [
    ["L", "mail"],
    ["l", "flag:starred", "mail"],
    ["e", "def456...", "", "labelled"]
  ],
  "content": ""
}
```

### Restore Email from Trash

Publish a deletion request for the gift wrap of the `folder:trash` label event:

```json
{
  "kind": 5,
  "tags": [
    ["e", "<trash_label_gift_wrap_id>"],
    ["k", "1059"]
  ],
  "content": ""
}
```

## Default States

When an email has no associated label events:

| Property | Default State |
|----------|---------------|
| Folder | Inbox |
| Read state | Unread |
| Starred | Not starred |
| Important | Not important |
