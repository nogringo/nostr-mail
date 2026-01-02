---
label: Plugins
icon: plug
order: 20
---

# Plugin System

The bridge supports an extensible plugin system for filtering emails before processing.

---

## How Plugins Work

Plugins are executables that communicate via JSON over stdin/stdout.

```
┌─────────────┐      JSON stdin      ┌────────┐      JSON stdout     ┌─────────────┐
│   Bridge    │ ──────────────────►  │ Plugin │ ──────────────────►  │   Bridge    │
└─────────────┘                      └────────┘                      └─────────────┘
```

---

## Plugin Input

The bridge sends email data to the plugin:

```json
{
  "type": "inbound",
  "event": {
    "from": "sender@example.com",
    "to": "recipient@example.com",
    "subject": "Hello",
    "body": "Message content",
    "senderPubkey": "abc123...",
    "recipientPubkey": "def456..."
  },
  "receivedAt": 1704067200,
  "sourceType": "smtp",
  "sourceInfo": "127.0.0.1"
}
```

---

## Plugin Output

The plugin returns a decision:

```json
{
  "id": "abc123...",
  "action": "accept",
  "msg": "Optional message"
}
```

### Actions

| Action | Description |
|--------|-------------|
| `accept` | Allow the email to be processed |
| `reject` | Reject the email (sender may be notified) |
| `shadowReject` | Silently discard (no notification) |

---

## Configuration

Set the plugin path in your environment:

```bash
PLUGIN_PATH=/path/to/your/plugin
```

---

## Available Plugins

### uid_ovh

A Dart plugin with advanced filtering:

- **NIP-51 Whitelist**: Only accept emails from whitelisted pubkeys
- **NIP-85 Trust Score**: Filter based on Web of Trust scores

#### Setup

```bash
cd bridge-plugins/uid_ovh
dart pub get
dart compile exe bin/uid_ovh.dart -o build/uid_ovh
```

#### Configuration

```bash
export PLUGIN_PATH=/path/to/build/uid_ovh
```

---

## Writing Your Own Plugin

### Requirements

1. Read JSON from stdin
2. Process the email data
3. Write JSON response to stdout
4. Exit with code 0

### Example (Node.js)

```javascript
#!/usr/bin/env node

const readline = require('readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  terminal: false
});

let input = '';

rl.on('line', (line) => {
  input += line;
});

rl.on('close', () => {
  const data = JSON.parse(input);

  // Your filtering logic here
  const isSpam = data.event.subject.toLowerCase().includes('viagra');

  const response = {
    id: data.event.senderPubkey,
    action: isSpam ? 'shadowReject' : 'accept',
    msg: isSpam ? 'Spam detected' : 'OK'
  };

  console.log(JSON.stringify(response));
});
```

### Example (Dart)

```dart
import 'dart:convert';
import 'dart:io';

void main() async {
  final input = await stdin.transform(utf8.decoder).join();
  final data = jsonDecode(input);

  // Your filtering logic here
  final isSpam = (data['event']['subject'] as String)
      .toLowerCase()
      .contains('viagra');

  final response = {
    'id': data['event']['senderPubkey'],
    'action': isSpam ? 'shadowReject' : 'accept',
    'msg': isSpam ? 'Spam detected' : 'OK',
  };

  print(jsonEncode(response));
}
```

---

## Web of Trust Filtering

The uid_ovh plugin supports NIP-85 trust scores:

1. Looks up sender's trust score on the network
2. Checks if sender is in recipient's NIP-51 whitelist
3. Applies configurable trust thresholds

```
Trust Score >= Threshold → Accept
Trust Score < Threshold → Reject
In Whitelist → Always Accept
```
