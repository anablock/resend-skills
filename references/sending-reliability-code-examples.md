# Sending Reliability — Code Examples

> These TypeScript examples are proposed additions to
> `references/sending-reliability.md`.

---

## Idempotency Key — Deterministic Generation

Generate a stable key tied to the logical event, not the request attempt.
Reuse the **exact same key** on every retry — this is what makes the
operation safe to retry.

```typescript
import { Resend } from 'resend';
import { createHash } from 'crypto';

const resend = new Resend(process.env.RESEND_API_KEY);

async function sendOrderConfirmation(orderId: string, to: string) {
  // Key is deterministic: same order always produces the same key
  const idempotencyKey = createHash('sha256')
    .update(`order-confirmation:${orderId}`)
    .digest('hex');

  const { data, error } = await resend.emails.send(
    {
      from: 'orders@yourdomain.com',
      to,
      subject: `Order #${orderId} confirmed`,
      html: `<p>Your order <strong>#${orderId}</strong> is confirmed.</p>`,
    },
    { idempotencyKey }
  );

  if (error) throw error;
  return data;
}
```

**Key principle:** Never use `Date.now()`, `Math.random()`, or a UUID
generated at call time as an idempotency key — these change on every
attempt and defeat the purpose.

---

## Retry with Exponential Backoff + Jitter

```typescript
async function sendWithRetry(
  payload: Parameters<typeof resend.emails.send>[0],
  maxAttempts = 4
) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const { data, error } = await resend.emails.send(payload);
      if (error) throw error;
      return data;
    } catch (err: any) {
      const isRetryable =
        err?.statusCode === 429 || // rate limited
        err?.statusCode >= 500;    // server error

      if (!isRetryable || attempt === maxAttempts) throw err;

      // Exponential backoff: 1s → 2s → 4s + random jitter up to 500ms
      const base = Math.pow(2, attempt - 1) * 1000;
      const jitter = Math.random() * 500;
      await new Promise((r) => setTimeout(r, base + jitter));
    }
  }
}
```

**Key principle:** Only retry on `429` (rate limit) and `5xx` (server
error). Never retry `4xx` client errors — a bad request or invalid email
address will not succeed on retry.

---

## Webhook Signature Verification + Idempotent Processing

Always verify Resend's webhook signature before processing. Always
deduplicate by event ID — webhooks can be delivered more than once.

```typescript
import { Resend } from 'resend';
import type { NextApiRequest, NextApiResponse } from 'next';

const resend = new Resend(process.env.RESEND_API_KEY);
const webhookSecret = process.env.RESEND_WEBHOOK_SECRET!;

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // Step 1: Verify the webhook signature
  let event;
  try {
    event = await resend.webhooks.constructEvent(
      JSON.stringify(req.body),
      {
        'svix-id': req.headers['svix-id'] as string,
        'svix-timestamp': req.headers['svix-timestamp'] as string,
        'svix-signature': req.headers['svix-signature'] as string,
      },
      webhookSecret
    );
  } catch {
    return res.status(400).json({ error: 'Invalid webhook signature' });
  }

  // Step 2: Deduplicate — webhooks can be delivered more than once
  const alreadyProcessed = await db.webhookEvents.findUnique({
    where: { id: event.data.email_id },
  });
  if (alreadyProcessed) return res.status(200).json({ ok: true });

  await db.webhookEvents.create({
    data: { id: event.data.email_id, type: event.type },
  });

  // Step 3: Handle the event
  switch (event.type) {
    case 'email.bounced':
      await suppressEmail(event.data.to[0]);
      break;
    case 'email.complained':
      await unsubscribeEmail(event.data.to[0]);
      break;
    case 'email.delivery_delayed':
      // Log for monitoring; do not retry manually — Resend retries automatically
      console.warn('Delivery delayed:', event.data.email_id);
      break;
  }

  return res.status(200).json({ ok: true });
}
```
