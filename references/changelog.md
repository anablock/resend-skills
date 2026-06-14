# Changelog

All notable changes to the Email Best Practices skill are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Planned
- LGPD (Brazil) compliance guidance
- India DPDP Act coverage
- AI-generated email content disclosure guidance

---

## [1.1.0] — 2025-06-14

### Added
- `references/bulk-sender-requirements.md` — Google & Yahoo 2024 mandatory
  requirements: SPF+DKIM both required, DMARC `p=none` minimum, one-click
  unsubscribe (RFC 8058), spam rate thresholds, BIMI setup
- `references/changelog.md` — this file
- `tests/scenarios/06-bulk-sender-compliance.md` — test scenario for the
  2024 bulk sender requirements

### Updated
- `references/compliance.md` — added Google/Yahoo 2024 enforcement dates,
  updated unsubscribe honour window to 2 days for bulk senders,
  added spam rate threshold table, added emerging regulations table
- `references/sending-reliability.md` — added TypeScript code examples for
  idempotency key generation, exponential backoff retry, and webhook
  signature verification with idempotent processing
- `SKILL.md` — added routing entry for `bulk-sender-requirements.md`
  and `changelog.md`

---

## [1.0.0] — 2024-01-01

### Added
- Initial release with full reference library:
  `deliverability`, `transactional-emails`, `transactional-email-catalog`,
  `marketing-emails`, `email-capture`, `email-types`, `compliance`,
  `sending-reliability`, `webhooks-events`, `list-management`, `accessibility`
- Test scenarios: spam/deliverability, multi-region compliance,
  retry/idempotency, webhook bounce handling, new SaaS email plan
