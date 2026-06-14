# Bulk Sender Requirements (Google & Yahoo 2024)

As of February–June 2024, Google and Yahoo enforce mandatory technical
requirements for senders exceeding **5,000 messages/day** to personal
inboxes. Non-compliance results in deferrals, spam filtering, or blocking.

---

## Who This Applies To

Any domain sending 5,000+ emails/day to Gmail or Yahoo personal accounts.
Applies to both transactional and marketing email.

---

## Required: SPF + DKIM (Both, Not Either)

All bulk senders must authenticate with **both** SPF and DKIM — not just one.

```dns
# SPF — authorise your sending IP
v=spf1 include:amazonses.com ~all

# DKIM — sign outbound messages (configure in your ESP)
# Minimum key length: 1024-bit (2048-bit recommended)
```

The `From:` header domain must **align** with either the SPF or DKIM domain
to pass DMARC checks.

---

## Required: DMARC Policy

Publish a DMARC record for your `From:` domain. Minimum: `p=none`.

```dns
_dmarc.yourdomain.com TXT "v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com"
```

| Policy          | Effect                                          | When to Use               |
|-----------------|-------------------------------------------------|---------------------------|
| `p=none`        | Monitor only — no enforcement                   | Starting out              |
| `p=quarantine`  | Suspicious mail goes to spam                    | After validating SPF/DKIM |
| `p=reject`      | Unauthenticated mail is rejected outright       | Full enforcement (required for BIMI) |

**Recommended path:** Start at `p=none`, monitor the `rua` aggregate reports
for 2–4 weeks, then advance to `p=quarantine`, then `p=reject`.

---

## Required: One-Click Unsubscribe (RFC 8058)

All marketing and subscribed-content emails must include:

1. A visible unsubscribe link in the email body/footer
2. The `List-Unsubscribe` and `List-Unsubscribe-Post` headers

```
List-Unsubscribe: <https://yourdomain.com/unsubscribe?token=abc123>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

Unsubscribe requests must be honoured within **2 days** (not 10 — stricter
than CAN-SPAM's 10-business-day window).

> **Note:** Transactional emails (password resets, receipts, OTPs) are
> exempt from the unsubscribe header requirement.

---

## Required: Spam Rate Threshold

| Threshold      | Consequence                                       |
|----------------|---------------------------------------------------|
| < 0.10%        | Safe zone                                         |
| 0.10–0.30%     | Warning territory — monitor closely               |
| > 0.30%        | Delivery impacted — immediate action required     |

Monitor your spam rate in **Google Postmaster Tools** and the
**Yahoo Complaint Feedback Loop (CFL)**.

---

## Recommended: BIMI (Brand Indicators for Message Identification)

BIMI displays your brand logo next to emails in Gmail and Yahoo.

**Prerequisites:**
- DMARC at `p=quarantine` or `p=reject` (not `p=none`)
- A square SVG logo hosted at a stable HTTPS URL
- (Optional) A Verified Mark Certificate (VMC) for the blue checkmark

```dns
default._bimi.yourdomain.com TXT "v=BIMI1; l=https://yourdomain.com/logo.svg;"
```

---

## Checklist

- [ ] SPF record published and valid
- [ ] DKIM signing enabled (2048-bit key)
- [ ] DMARC record published (`p=none` minimum)
- [ ] `List-Unsubscribe` + `List-Unsubscribe-Post` headers on all marketing email
- [ ] Unsubscribes honoured within 2 days
- [ ] Spam rate monitored and below 0.10%
- [ ] Google Postmaster Tools domain verified
- [ ] Yahoo CFL enrolled
