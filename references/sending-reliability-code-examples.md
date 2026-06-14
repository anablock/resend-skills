# Sending Reliability — Code Examples

> TypeScript examples for Anablock's three primary email surfaces:
> **Anablock CRM** (outreach sequences), **Anablock Echo** (AI-triggered
> communications), and **Sterolux** (e-commerce transactional + marketing).
> Proposed additions to `references/sending-reliability.md`.

---

## 1. Idempotency Keys — Anablock Patterns

Generate a stable key tied to the **logical event**, not the request attempt.
Reuse the exact same key on every retry.

### Anablock CRM — Outreach Sequence Email

```typescript
import { Resend } from 'resend';
import { createHash } from 'crypto';

const resend = new Resend(process.env.RESEND_API_KEY);

/**
 * Send a CRM outreach sequence step.
 * Key is scoped to: contactId + sequenceId + stepNumber
 * Same contact can be in multiple sequences; same step is never sent twice.
 */
async function sendCrmSequenceStep({
  contactId,
  sequenceId,
  stepNumber,
  to,
  subject,
  html,
}: {
  contactId: string;
  sequenceId: string;
  stepNumber: number;
  to: string;
  subject: string;
  html: string;
}) {
  const idempotencyKey = createHash('sha256')
    .update(`crm-sequence:${contactId}:${sequenceId}:step-${stepNumber}`)
    .digest('hex');

  const { data, error } = await resend.emails.send(
    {
      from: 'outreach@anablock.com',
      to,
      subject,
      html,
      headers: {
        'List-Unsubscribe': `<https://anablock.com/unsubscribe?contactId=${contactId}>`,
        'List-Unsubscribe-Post': 'List-Unsubscribe=One-Click',
      },
    },
    { idempotencyKey }
  );

  if (error) throw error;
  return data;
}
```

### Anablock Echo — Appointment Reminder

```typescript
/**
 * Send an Echo appointment reminder.
 * Key is scoped to: appointmentId + reminderType (24h, 1h, etc.)
 * Prevents duplicate reminders if Echo retries after a transient failure.
 */
async function sendEchoAppointmentReminder({
  appointmentId,
  reminderType,
  to,
  patientName,
  appointmentTime,
  practicePhone,
}: {
  appointmentId: string;
  reminderType: '24h' | '1h' | '15min';
  to: string;
  patientName: string;
  appointmentTime: string;
  practicePhone: string;
}) {
  const idempotencyKey = createHash('sha256')
    .update(`echo-reminder:${appointmentId}:${reminderType}`)
    .digest('hex');

  const { data, error } = await resend.emails.send(
    {
      from: 'reminders@anablock.com',
      to,
      subject: `Appointment reminder — ${appointmentTime}`,
      html: `
        <p>Hi ${patientName},</p>
        <p>This is a reminder of your upcoming appointment at <strong>${appointmentTime}</strong>.</p>
        <p>To reschedule or cancel, call us at ${practicePhone} or reply to this email.</p>
      `,
      // Transactional — no List-Unsubscribe header required
    },
    { idempotencyKey }
  );

  if (error) throw error;
  return data;
}
```

### Sterolux — Order Confirmation

```typescript
/**
 * Send a Sterolux order confirmation.
 * Key is scoped to: orderId
 * Guarantees exactly-once delivery even if the checkout webhook fires twice.
 */
async function sendSteroluxOrderConfirmation({
  orderId,
  to,
  customerName,
  orderTotal,
  estimatedDelivery,
}: {
  orderId: string;
  to: string;
  customerName: string;
  orderTotal: string;
  estimatedDelivery: string;
}) {
  const idempotencyKey = createHash('sha256')
    .update(`sterolux-order-confirmation:${orderId}`)
    .digest('hex');

  const { data, error } = await resend.emails.send(
    {
      from: 'orders@sterolux.com',
      to,
      subject: `Order #${orderId} confirmed — Sterolux`,
      html: `
        <p>Hi ${customerName},</p>
        <p>Thank you for your order. Here are your details:</p>
        <ul>
          <li><strong>Order ID:</strong> #${orderId}</li>
          <li><strong>Total:</strong> ${orderTotal}</li>
          <li><strong>Estimated delivery:</strong> ${estimatedDelivery}</li>
        </ul>
        <p>Questions? Reply to this email or visit <a href="https://sterolux.com/support">sterolux.com/support</a>.</p>
      `,
      // Transactional — no List-Unsubscribe header required
    },
    { idempotencyKey }
  );

  if (error) throw error;
  return data;
}
```

---

## 2. Retry with Exponential Backoff + Jitter

Shared utility for all Anablock sending surfaces. Only retry on `429`
(rate limit) and `5xx` (server error) — never on `4xx` client errors.

```typescript
async function sendWithRetry(
  payload: Parameters<typeof resend.emails.send>[0],
  options?: Parameters<typeof resend.emails.send>[1],
  maxAttempts = 4
) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const { data, error } = await resend.emails.send(payload, options);
      if (error) throw error;
      return data;
    } catch (err: any) {
      const isRetryable =
        err?.statusCode === 429 || // rate limited
        err?.statusCode >= 500;    // Resend server error

      if (!isRetryable || attempt === maxAttempts) throw err;

      // Exponential backoff: 1s → 2s → 4s + random jitter up to 500ms
      const base = Math.pow(2, attempt - 1) * 1000;
      const jitter = Math.random() * 500;
      await new Promise((r) => setTimeout(r, base + jitter));
    }
  }
}

// Usage — Anablock CRM sequence step with retry
await sendWithRetry(
  {
    from: 'outreach@anablock.com',
    to: contact.email,
    subject: 'Following up on your Anablock CRM trial',
    html: emailHtml,
  },
  { idempotencyKey }
);
```

---

## 3. Webhook Handler — Bounce & Complaint Processing

Anablock CRM and Echo must suppress bounced and complained addresses
immediately to protect sender reputation and maintain list hygiene.

```typescript
import { Resend } from 'resend';
import type { NextApiRequest, NextApiResponse } from 'next';

const resend = new Resend(process.env.RESEND_API_KEY);
const webhookSecret = process.env.RESEND_WEBHOOK_SECRET!;

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // Step 1: Verify Resend webhook signature
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

  // Step 2: Deduplicate — Resend may deliver the same event more than once
  const alreadyProcessed = await db.webhookEvents.findUnique({
    where: { id: event.data.email_id },
  });
  if (alreadyProcessed) return res.status(200).json({ ok: true });

  await db.webhookEvents.create({
    data: { id: event.data.email_id, type: event.type },
  });

  const recipientEmail = event.data.to[0];

  switch (event.type) {
    case 'email.bounced':
      // Hard bounce — suppress in both Anablock CRM and Resend
      await Promise.all([
        anablockCrm.contacts.suppress(recipientEmail),   // mark undeliverable in CRM
        resend.contacts.remove({ email: recipientEmail, audienceId: process.env.RESEND_AUDIENCE_ID! }),
      ]);
      break;

    case 'email.complained':
      // Spam complaint — unsubscribe immediately; 2-day window starts now
      await Promise.all([
        anablockCrm.contacts.unsubscribe(recipientEmail),
        resend.contacts.update({
          email: recipientEmail,
          audienceId: process.env.RESEND_AUDIENCE_ID!,
          unsubscribed: true,
        }),
      ]);
      break;

    case 'email.delivery_delayed':
      // Log for monitoring — Resend retries automatically; do not retry manually
      console.warn('[Anablock] Delivery delayed:', {
        emailId: event.data.email_id,
        to: recipientEmail,
      });
      break;

    case 'email.opened':
    case 'email.clicked':
      // Update CRM contact engagement score / last contacted date
      await anablockCrm.contacts.recordEngagement({
        email: recipientEmail,
        event: event.type,
        timestamp: new Date().toISOString(),
      });
      break;
  }

  return res.status(200).json({ ok: true });
}
```
