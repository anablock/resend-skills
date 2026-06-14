# Compliance Additions (2024+)

> These sections are proposed additions to `references/compliance.md`.
> They cover the Google/Yahoo 2024 bulk sender mandates and an emerging
> regulations watch list not present in the original file.

---

## Google & Yahoo 2024 Bulk Sender Requirements

Enforced from February 2024 (authentication) and June 2024 (unsubscribe
headers) for senders of **5,000+ messages/day** to personal Gmail/Yahoo inboxes.

| Requirement              | Specification                                              |
|--------------------------|------------------------------------------------------------|
| SPF + DKIM               | **Both** required (not either/or)                          |
| DMARC                    | `p=none` minimum; `From:` domain alignment required        |
| List-Unsubscribe header  | RFC 8058 one-click header on all marketing email           |
| Unsubscribe honour time  | **2 days** (stricter than CAN-SPAM's 10 business days)     |
| Spam rate                | Must stay below **0.30%**; target below **0.10%**          |
| PTR records              | Valid forward/reverse DNS for all sending IPs              |

> See `references/bulk-sender-requirements.md` for full setup guidance
> and DNS record examples.

### Unsubscribe Honour Window Note

> **Note for bulk senders:** Google and Yahoo require unsubscribe requests
> to be honoured within **2 days**, which is stricter than CAN-SPAM's
> 10-business-day window. If you send 5,000+ emails/day, the 2-day rule
> applies regardless of your recipients' location.

---

## Emerging Regulations (Watch List)

| Regulation         | Jurisdiction  | Status           | Key Requirement                       |
|--------------------|---------------|------------------|---------------------------------------|
| LGPD               | Brazil        | In force         | Consent-based, similar to GDPR        |
| DPDP Act           | India         | Enacted 2023     | Explicit consent, data localisation   |
| Law 25 (Bill 64)   | Quebec, CA    | In force 2023    | Stricter than CASL; opt-in required   |
| ePrivacy Reg.      | EU            | Pending          | Will replace ePrivacy Directive       |

**Recommendation:** For global senders, apply the strictest applicable
standard across your entire list rather than segmenting by jurisdiction —
it simplifies compliance and future-proofs your programme.
