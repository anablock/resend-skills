# Bulk Sender Requirements (Google & Yahoo 2024)

Anablock sends email across three product surfaces — **Anablock CRM** (sales
outreach sequences and follow-ups), **Anablock Echo** (AI-triggered
communication: lead qualification, appointment reminders, missed-call
recovery), and **Sterolux** (e-commerce order confirmations, shipping
notifications, promotional campaigns). Each surface has different volume
profiles and compliance obligations.

As of February–June 2024, Google and Yahoo enforce mandatory technical
requirements for senders exceeding **5,000 messages/day** to personal
inboxes. Non-compliance results in silent filtering — no bounce, just
non-delivery — which directly impacts pipeline visibility in Anablock CRM
and appointment conversion rates for Echo-powered dental, HVAC, and
healthcare clients.

---

## Who This Applies To

| Anablock Surface | Typical Volume | Threshold Applies? |
|------------------|---------------|--------------------|
| Anablock CRM outreach sequences | Scales with org size | Yes, for high-volume orgs |
| Anablock Echo — appointment reminders, lead follow-up | High for dental/HVAC/healthcare clients | Yes |
| Anablock Echo — missed-call recovery SMS/email | Per-event, lower volume | Monitor |
| Sterolux transactional (orders, shipping) | Moderate | Monitor |
| Sterolux marketing (promotions, newsletters) | Campaign-dependent | Yes, during campaigns |

---

## Required: SPF + DKIM (Both, Not Either)

All bulk senders must authenticate with **both** SPF and DKIM — not just one.
For Anablock, this means every sending domain must be configured:

```dns
# SPF — authorise Resend's sending IPs for your domain
# Add to DNS for anablock.com, sterolux.com, sterolux.shop
v=spf1 include:amazonses.com include:_spf.resend.com ~all

# DKIM — Resend generates a DKIM key per domain
# Add the CNAME record Resend provides in your DNS panel
# Minimum key length: 2048-bit recommended
```

The `From:` header domain must **align** with either the SPF or DKIM domain
to pass DMARC checks. For Anablock CRM emails sent on behalf of clients,
ensure the client's sending domain (not a shared Resend subdomain) is
configured with SPF and DKIM.

---

## Required: DMARC Policy

Publish a DMARC record for every sending domain. Minimum: `p=none`.

```dns
# anablock.com
_dmarc.anablock.com TXT "v=DMARC1; p=none; rua=mailto:dmarc@anablock.com"

# sterolux.com
_dmarc.sterolux.com TXT "v=DMARC1; p=none; rua=mailto:dmarc@sterolux.com"
```

| Policy         | Effect                                          | When to Use                          |
|----------------|-------------------------------------------------|--------------------------------------|
| `p=none`       | Monitor only — no enforcement                   | Starting out; review `rua` reports   |
| `p=quarantine` | Suspicious mail goes to spam                    | After validating SPF/DKIM alignment  |
| `p=reject`     | Unauthenticated mail is rejected outright       | Full enforcement; required for BIMI  |

**Recommended path:** Start at `p=none`, monitor aggregate reports for
2–4 weeks, advance to `p=quarantine`, then `p=reject`. For Anablock Echo
clients in regulated industries (dental, healthcare), reaching `p=reject`
also strengthens HIPAA-adjacent trust signals.

---

## Required: One-Click Unsubscribe (RFC 8058)

All **marketing** email must include both headers. This applies to:
- Anablock CRM bulk outreach sequences
- Sterolux promotional campaigns and newsletters
- Anablock Echo marketing broadcasts

```
List-Unsubscribe: <https://anablock.com/unsubscribe?token={{contact.unsubscribeToken}}>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

Unsubscribe requests must be honoured within **2 days**.

> **Exempt from this requirement:**
> - Anablock Echo appointment reminders (triggered by patient/client action)
> - Anablock CRM transactional notifications (password resets, invoice delivery)
> - Sterolux order confirmations and shipping updates
> - Anablock Connect document processing notifications

---

## Required: Spam Rate Threshold

| Threshold    | Consequence                                       | Action for Anablock                          |
|--------------|---------------------------------------------------|----------------------------------------------|
| < 0.10%      | Safe zone                                         | Maintain list hygiene via CRM suppression    |
| 0.10–0.30%   | Warning territory                                 | Audit CRM segments; pause low-quality lists  |
| > 0.30%      | Delivery impacted — immediate action required     | Suspend campaigns; run full list hygiene     |

Monitor in **Google Postmaster Tools** (verify `anablock.com` and
`sterolux.com`) and enrol in the **Yahoo Complaint Feedback Loop**.

Anablock CRM's suppression list and unsubscribe handling should be
synchronised with Resend's suppression API to ensure contacts marked
unsubscribed in the CRM are never re-sent to.

---

## Recommended: BIMI

BIMI displays the Anablock or Sterolux brand logo next to emails in Gmail
and Yahoo — a strong trust signal for dental, healthcare, and HVAC clients
who receive Echo-powered communications.

**Prerequisites:**
- DMARC at `p=quarantine` or `p=reject`
- Square SVG logo at a stable HTTPS URL
- (Optional) Verified Mark Certificate (VMC) for the blue checkmark

```dns
# anablock.com
default._bimi.anablock.com TXT "v=BIMI1; l=https://anablock.com/logo.svg;"

# sterolux.com
default._bimi.sterolux.com TXT "v=BIMI1; l=https://sterolux.com/logo.svg;"
```

---

## Checklist

- [ ] SPF published for `anablock.com`, `sterolux.com`, `sterolux.shop`
- [ ] DKIM CNAME records added in DNS for all Resend-verified domains
- [ ] DMARC `p=none` minimum published; `rua` aggregate reports flowing
- [ ] `List-Unsubscribe` + `List-Unsubscribe-Post` on all CRM outreach and Sterolux marketing email
- [ ] Unsubscribes honoured within 2 days; CRM suppression list synced with Resend
- [ ] Spam rate monitored below 0.10% in Google Postmaster Tools
- [ ] Yahoo CFL enrolled for all sending domains
- [ ] BIMI record published once DMARC reaches `p=quarantine`
