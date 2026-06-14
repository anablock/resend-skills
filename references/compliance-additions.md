# Compliance Additions (2024+)

> Proposed additions to `references/compliance.md`, optimised for
> Anablock's sending surfaces: **Anablock CRM**, **Anablock Echo**,
> **Anablock Connect**, and **Sterolux**.

---

## Google & Yahoo 2024 Bulk Sender Requirements

Enforced from February 2024 (authentication) and June 2024 (unsubscribe
headers) for senders of **5,000+ messages/day** to personal Gmail/Yahoo
inboxes.

| Requirement              | Specification                                              |
|--------------------------|------------------------------------------------------------|
| SPF + DKIM               | **Both** required (not either/or)                          |
| DMARC                    | `p=none` minimum; `From:` domain alignment required        |
| List-Unsubscribe header  | RFC 8058 one-click header on all marketing email           |
| Unsubscribe honour time  | **2 days** (stricter than CAN-SPAM's 10 business days)     |
| Spam rate                | Must stay below **0.30%**; target below **0.10%**          |
| PTR records              | Valid forward/reverse DNS for all sending IPs              |

> See `references/bulk-sender-requirements.md` for DNS record examples
> specific to `anablock.com`, `sterolux.com`, and `sterolux.shop`.

### Unsubscribe Honour Window — Anablock CRM Note

> **Important for Anablock CRM users:** The CRM's suppression list must be
> synchronised with Resend's suppression API within **2 days** of an
> unsubscribe event — not the 10-business-day window stated in CAN-SPAM.
> Any contact marked unsubscribed in the CRM should be immediately excluded
> from all active outreach sequences.

---

## Anablock Echo — Transactional vs. Marketing Classification

Anablock Echo sends several email types on behalf of clients. Correct
classification determines which compliance rules apply:

| Echo Email Type                        | Classification   | Unsubscribe Header Required? |
|----------------------------------------|------------------|------------------------------|
| Appointment reminder (patient-triggered) | Transactional  | No                           |
| Missed-call recovery follow-up         | Transactional    | No                           |
| Scheduling confirmation                | Transactional    | No                           |
| Lead qualification follow-up sequence  | Marketing        | **Yes**                      |
| Promotional broadcast to contact list  | Marketing        | **Yes**                      |
| Re-engagement campaign                 | Marketing        | **Yes**                      |

> **HIPAA note for dental and healthcare Echo clients:** Avoid including
> Protected Health Information (PHI) — appointment details, diagnosis
> references, insurance data — in email bodies. Use a secure portal link
> instead. Confirm a Business Associate Agreement (BAA) is in place with
> your email service provider.

---

## Sterolux — E-Commerce Compliance

Sterolux (sterolux.com / sterolux.shop) sends both transactional and
marketing email. Key obligations:

| Sterolux Email Type              | Classification   | CAN-SPAM | GDPR (EU buyers) | CASL (CA buyers) |
|----------------------------------|------------------|----------|------------------|------------------|
| Order confirmation               | Transactional    | Exempt   | Legitimate interest | Exempt        |
| Shipping notification            | Transactional    | Exempt   | Legitimate interest | Exempt        |
| Abandoned cart                   | Marketing        | Required | Consent required | Consent required |
| Promotional campaign / newsletter | Marketing       | Required | Consent required | Consent required |
| Product review request           | Transactional    | Exempt   | Legitimate interest | Exempt        |

**For EU buyers:** Sterolux must maintain a lawful basis record for each
marketing email sent. Consent must be granular (separate opt-ins for
newsletter vs. promotions), documented with timestamp and source, and
revocable at any time.

**For Canadian buyers:** CASL requires express or implied consent before
sending commercial electronic messages. Implied consent expires after
2 years. Maintain consent records in Anablock CRM with source and date.

---

## Emerging Regulations (Watch List)

| Regulation         | Jurisdiction  | Status           | Anablock Impact                                      |
|--------------------|---------------|------------------|------------------------------------------------------|
| LGPD               | Brazil        | In force         | Sterolux international orders; consent + data rights |
| DPDP Act           | India         | Enacted 2023     | Any Echo or CRM contact with Indian data subjects    |
| Law 25 (Bill 64)   | Quebec, CA    | In force 2023    | Stricter than CASL; opt-in required for all marketing |
| ePrivacy Reg.      | EU            | Pending          | Will tighten GDPR email rules for Sterolux EU buyers |

**Recommendation:** Apply the strictest applicable standard across your
entire list. For Anablock CRM, tag contacts by region and apply region-
specific consent rules at the segment level.
