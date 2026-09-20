---
label: Custom Bridge
icon: tools
order: 5
---

# Building a Custom Bridge

How to extend the bridge with custom inbound or outbound providers.

---

## Custom Outbound Provider

To add a new outbound provider (e.g., SendGrid, SES, etc.), create a new file in:

```
bridge-outbound/src/outbound/
```

### 1. Create the Provider

```typescript
// bridge-outbound/src/outbound/sendgrid.ts
import { OutboundProvider } from './interface.js';
import { OutgoingEmail } from '../types.js';
import { config } from '../config.js';

export class SendGridProvider implements OutboundProvider {
  name = 'sendgrid';

  async send(email: OutgoingEmail): Promise<void> {
    // email.to = recipient address
    // email.raw = RFC 2822 content

    // Your SendGrid logic here
    await sendgrid.send({
      to: email.to,
      raw: email.raw,
    });

    console.log(`Email sent via SendGrid to ${email.to}`);
  }
}
```

### 2. Register in index.ts

```typescript
// bridge-outbound/src/outbound/index.ts
import { OutboundProvider } from './interface.js';
import { MailgunProvider } from './mailgun.js';
import { SmtpProvider } from './smtp.js';
import { SendGridProvider } from './sendgrid.js';  // Add import
import { config } from '../config.js';

export function createOutboundProvider(): OutboundProvider {
  switch (config.outboundProvider) {
    case 'mailgun':
      return new MailgunProvider();
    case 'sendgrid':                    // Add case
      return new SendGridProvider();
    case 'smtp':
    default:
      return new SmtpProvider();
  }
}
```

### 3. Add Config (if needed)

```typescript
// bridge-outbound/src/config.ts
export const config = {
  // ... existing config
  sendgridApiKey: process.env.SENDGRID_API_KEY || '',
};
```

### Interface Reference

```typescript
interface OutboundProvider {
  name: string;
  send(email: OutgoingEmail): Promise<void>;
}

interface OutgoingEmail {
  to: string;   // Recipient email address
  raw: string;  // RFC 2822 email content
}
```

---

## Custom Inbound Provider

To add a new inbound provider (e.g., AWS SES, custom webhook), create a new file in:

```
bridge-inbound/
```

### 1. Create the Provider

```typescript
// bridge-inbound/ses/src/index.ts
import {
  initConfig,
  processIncomingEmail,
  IncomingEmail,
} from '@nostr-mail/bridge-core';

initConfig();

// Example: AWS SES webhook
export async function handleSesNotification(notification: any) {
  const email: IncomingEmail = {
    from: notification.mail.source,
    to: notification.mail.destination[0],
    subject: notification.mail.commonHeaders.subject,
    body: notification.content,
    rawContent: notification.content,
    timestamp: Math.floor(Date.now() / 1000),
  };

  // processIncomingEmail handles:
  // 1. Resolve the recipient pubkey
  // 2. Run the plugin filter
  // 3. Fetch DM relays
  // 4. Gift wrap and publish
  return processIncomingEmail(email, 'ses', 'aws');
}
```

Pass the recipient pubkey as a fourth argument when your source already knows
it, and the address lookup is skipped:

```typescript
processIncomingEmail(email, sourceType, sourceInfo, recipientPubkey?)
```

!!!warning Use the envelope
`email.to` should be the SMTP envelope recipient, not the `To:` header. A Bcc
recipient only ever appears in the envelope.
!!!

### @nostr-mail/bridge-core exports

```typescript
// Configuration
initConfig()
getConfig()

// The whole inbound path in one call
processIncomingEmail(email, sourceType, sourceInfo, recipientPubkey?)

// Individual steps
extractPubkeyFromEmail(address)  // Resolve an address to a pubkey
lookupNip05(name, domain)        // NIP-05 lookup
fetchDMRelays(pubkey)            // DM relays (kind 10050)
publishToRelays(event, relays)   // Publish
runPlugin(path, input)           // Run the filter plugin
getInboundPubkey()               // The bridge's own pubkey
```

The gift wrapping itself comes from the [`nostr-mail`](/sdk/javascript/) npm
package, which `bridge-core` re-exports as `NostrMailClient`.

### IncomingEmail Interface

```typescript
interface IncomingEmail {
  from: string;       // Sender address
  to: string;         // Recipient address
  subject: string;    // Email subject
  body: string;       // Email body
  rawContent: string; // Full RFC 2822 content
  timestamp: number;  // Unix timestamp
}
```
