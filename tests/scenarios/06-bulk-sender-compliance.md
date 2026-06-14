# Test Scenario 06 — Bulk Sender Compliance (Google & Yahoo 2024)

## Scenario

A B2B SaaS company is sending 8,000 emails/day — a mix of transactional
(password resets, invoices) and marketing (weekly newsletter, product
announcements). Their DevOps lead reports that ~3% of Gmail recipients
aren't receiving the newsletter. No bounce errors are returned by the API.

## User Prompt

> "We send about 8,000 emails a day. Gmail users aren't getting our
> newsletter but we're not seeing any bounces. What's wrong and how do
> we fix it?"

## Expected Agent Behaviour

1. **Identify the threshold:** 8,000/day exceeds the 5,000/day bulk sender
   threshold — Google's 2024 requirements apply.

2. **Diagnose the likely causes** (in priority order):
   - Missing or failing DMARC record (most common cause of silent filtering)
   - Missing `List-Unsubscribe` / `List-Unsubscribe-Post` headers on newsletter
   - Spam rate above 0.10% in Google Postmaster Tools
   - SPF/DKIM misalignment between `From:` domain and sending domain

3. **Prescribe the fix:**
   - Verify DMARC: `dig TXT _dmarc.yourdomain.com`
   - Check Google Postmaster Tools for spam rate and domain reputation
   - Confirm `List-Unsubscribe` header is present on newsletter sends
   - Confirm both SPF and DKIM pass (not just one)

4. **Separate transactional from marketing:**
   - Transactional email (password resets, invoices) → exempt from
     unsubscribe header requirement
   - Marketing email (newsletter, announcements) → must include
     `List-Unsubscribe` + `List-Unsubscribe-Post` headers

5. **Recommend ongoing monitoring:**
   - Enrol in Yahoo Complaint Feedback Loop
   - Set up DMARC aggregate report (`rua`) parsing
   - Target spam rate < 0.10%

## Expected Output (Summary)

The agent should produce:
- A prioritised checklist of DNS/header fixes
- DNS record examples for DMARC, SPF, DKIM
- The `List-Unsubscribe` + `List-Unsubscribe-Post` header snippet
- A note distinguishing transactional (exempt) vs. marketing (required) obligations
- References to Google Postmaster Tools and Yahoo CFL enrolment

## Pass Criteria

- [ ] Agent correctly identifies the 5,000/day threshold as the trigger
- [ ] Agent recommends checking DMARC as the first diagnostic step
- [ ] Agent provides the `List-Unsubscribe-Post: List-Unsubscribe=One-Click` header
- [ ] Agent distinguishes transactional (exempt) from marketing (required)
- [ ] Agent recommends Google Postmaster Tools for spam rate monitoring
- [ ] Agent specifies the 2-day unsubscribe honour window (not 10 business days)
- [ ] Agent does NOT recommend buying a new domain as the first fix
- [ ] Agent does NOT suggest the issue is a bounce (correctly identifies silent filtering)

## Related References

- `references/bulk-sender-requirements.md` — full technical requirements
- `references/compliance.md` — regulatory context
- `references/deliverability.md` — SPF/DKIM/DMARC setup
