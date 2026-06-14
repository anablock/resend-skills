# Test Scenario 06 — Bulk Sender Compliance (Google & Yahoo 2024)

## Scenario

An Anablock Echo client — a dental practice with 3 locations — is sending
~8,500 emails/day across appointment reminders, lead qualification follow-ups,
and a monthly patient newsletter. Their office manager reports that patients
with Gmail addresses stopped receiving the monthly newsletter two weeks ago.
No bounce errors are returned by the Resend API. Appointment reminder
delivery appears unaffected.

## User Prompt

> "Our dental practice uses Anablock Echo and we send around 8,500 emails
> a day. Gmail patients aren't getting our monthly newsletter but we're
> not seeing any bounces in the dashboard. Appointment reminders seem fine.
> What's wrong and how do we fix it?"

## Expected Agent Behaviour

1. **Identify the threshold:** 8,500/day exceeds the 5,000/day bulk sender
   threshold — Google's 2024 requirements apply to this Echo client.

2. **Explain the symptom:** Silent filtering (no bounce, just non-delivery)
   is the hallmark of a DMARC or unsubscribe header failure, not a
   deliverability issue with the sending IP. Appointment reminders are
   unaffected because they are transactional and exempt from the
   unsubscribe header requirement.

3. **Diagnose the likely causes** (in priority order):
   - Missing or failing DMARC record on the practice's sending domain
   - Missing `List-Unsubscribe` / `List-Unsubscribe-Post` headers on the
     monthly newsletter (marketing email — headers required)
   - Spam rate above 0.10% in Google Postmaster Tools
   - SPF/DKIM misalignment between `From:` domain and Resend sending domain

4. **Prescribe the fix:**
   - Verify DMARC: `dig TXT _dmarc.theirpractice.com`
   - Check Google Postmaster Tools for spam rate and domain reputation
   - Confirm `List-Unsubscribe` + `List-Unsubscribe-Post` headers are
     present on the newsletter send (not on appointment reminders)
   - Confirm both SPF and DKIM pass via MXToolbox or mail-tester.com

5. **Distinguish Echo email types:**
   - Appointment reminders → transactional → exempt from unsubscribe headers
   - Monthly newsletter → marketing → **must** include unsubscribe headers
   - Lead qualification follow-ups → marketing → **must** include unsubscribe headers

6. **HIPAA note:** Remind the client not to include PHI (appointment
   details, diagnosis, insurance info) in email bodies — use a secure
   portal link instead.

7. **Recommend ongoing monitoring:**
   - Verify practice domain in Google Postmaster Tools
   - Enrol in Yahoo Complaint Feedback Loop
   - Set up DMARC `rua` aggregate report parsing
   - Target spam rate < 0.10%
   - Sync unsubscribes from Resend back to Anablock CRM within 2 days

## Expected Output (Summary)

The agent should produce:
- A prioritised checklist of DNS/header fixes for the dental practice domain
- DNS record examples for DMARC, SPF, DKIM
- The `List-Unsubscribe` + `List-Unsubscribe-Post` header snippet
- A table distinguishing Echo email types (transactional vs. marketing)
- A HIPAA note about PHI in email bodies
- References to Google Postmaster Tools and Yahoo CFL enrolment

## Pass Criteria

- [ ] Agent correctly identifies the 5,000/day threshold as the trigger
- [ ] Agent explains silent filtering vs. bounce (no API error ≠ delivered)
- [ ] Agent recommends checking DMARC as the first diagnostic step
- [ ] Agent provides the `List-Unsubscribe-Post: List-Unsubscribe=One-Click` header
- [ ] Agent correctly classifies appointment reminders as transactional (exempt)
- [ ] Agent correctly classifies the newsletter as marketing (headers required)
- [ ] Agent specifies the 2-day unsubscribe honour window
- [ ] Agent recommends Google Postmaster Tools for spam rate monitoring
- [ ] Agent includes a HIPAA note about PHI in email bodies
- [ ] Agent does NOT recommend buying a new domain as the first fix
- [ ] Agent does NOT suggest the issue is a hard bounce

## Related References

- `references/bulk-sender-requirements.md` — full technical requirements
- `references/compliance-additions.md` — Echo transactional vs. marketing
  classification table and HIPAA note
- `references/deliverability.md` — SPF/DKIM/DMARC setup
- `references/webhooks-events.md` — syncing Resend bounce/complaint events
  back to Anablock CRM
