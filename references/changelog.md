# Changelog

All notable changes to the Anablock email best practices skill are documented
here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

This skill covers email sent across Anablock's product surfaces:
- **Anablock CRM** — sales outreach, follow-up sequences, pipeline notifications
- **Anablock Echo** — AI-triggered communications: appointment reminders,
  lead qualification, missed-call recovery, scheduling confirmations
- **Anablock Connect** — document processing notifications, extraction alerts
- **Sterolux** — e-commerce transactional (orders, shipping) and marketing
  (promotions, newsletters) for sterolux.com and sterolux.shop

---

## [Unreleased]

### Planned
- HIPAA-adjacent email handling guidance for Anablock Echo dental and
  healthcare clients (PHI in email bodies, BAA considerations)
- Anablock Connect document notification templates (extraction complete,
  validation failed, routing confirmation)
- Sterolux abandoned cart email sequence best practices
- Multi-domain sending strategy for anablock.com vs. sterolux.com vs.
  sterolux.shop
- LGPD (Brazil) compliance guidance for Sterolux international orders
- India DPDP Act coverage

---

## [1.1.0] — 2025-06-14

### Added
- `references/bulk-sender-requirements.md` — Google & Yahoo 2024 mandatory
  requirements, optimised for Anablock CRM, Echo, and Sterolux sending surfaces
- `references/changelog.md` — this file
- `tests/scenarios/06-bulk-sender-compliance.md` — test scenario using an
  Anablock Echo dental client as the example sender

### Updated
- `references/compliance-additions.md` — Google/Yahoo 2024 enforcement,
  2-day unsubscribe window, emerging regulations table; all examples
  reference anablock.com and sterolux.com domains
- `references/sending-reliability-code-examples.md` — TypeScript examples
  using Anablock CRM contact IDs, Echo appointment events, and Sterolux
  order IDs as idempotency key sources

---

## [1.0.0] — 2024-01-01

### Added
- Initial release with full reference library:
  `deliverability`, `transactional-emails`, `transactional-email-catalog`,
  `marketing-emails`, `email-capture`, `email-types`, `compliance`,
  `sending-reliability`, `webhooks-events`, `list-management`, `accessibility`
- Test scenarios: spam/deliverability, multi-region compliance,
  retry/idempotency, webhook bounce handling, new SaaS email plan
