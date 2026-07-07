---
name: syntance-legal-pl-eu
description: PL/EU legal compliance - European Accessibility Act, GDPR (data subject rights, 72h breach notification), Digital Services Act, Polish Consumer Rights Act, cookie consent (Polish Telecom Law art. 173), Omnibus Directive, e-services law, DPAs with processors, mandatory footer/imprint info. Activate when working on the footer, checkout, consent forms, cookie banners, legal pages, or the accessibility statement.
---

# PL/EU Legal Compliance

## EAA — European Accessibility Act (mandatory)
- Scope: all B2C digital products in the EU. Requirement: **WCAG 2.2 AA** (→ `syntance-perf-a11y-seo-geo`). Poland penalty ~80,000 PLN + sales ban.
- `/accessibility-statement` page: compliance status, last review date, assessment method, accessibility coordinator contact, link to the national DPA/Ombudsman complaint procedure.
- "Report an accessibility issue" button in the footer. Annual audit (axe + Wave + manual screen reader test), fix critical issues within 30 days.

## GDPR — Data Subject Rights (DSR)
- Access (Art. 15), rectification (Art. 16), erasure (Art. 17), restriction (Art. 18), portability (Art. 20), objection (Art. 21).
- Implementation: `/account/privacy` with download/delete/restrict/export CTAs + a webhook to all processors (Sentry, PostHog, Resend, MailerLite).
- Retention: invoices 5 years (Polish tax law), security logs 12 months (legitimate interest).
- **72h breach notification**: risk assessment within 24h → notify the national data protection authority within 72h → notify affected users when risk is high. Runbook at `docs/runbook/data-breach.md`.

## Cookies (Polish Telecom Law art. 173, implementing ePrivacy)
- Consent must be **opt-in** (never pre-checked). Three equally prominent buttons: **Accept / Reject / Customize**. Never use "X" as a dismiss action.
- The banner must not block scrolling. Consent gate: GA/Meta/PostHog/Hotjar load ONLY after consent (+ Partytown, Consent Mode v2).
- Tooling: **Klaro** (open source) or Cookiebot. Categories: Necessary/Analytics/Marketing/Preferences. Explicit retention periods.

## Polish Consumer Rights Act (B2C)
- 14-day right of withdrawal (form + email sent after shipment). Business identification info (name, address, tax/registration IDs, contact).
- Terms of Service + Privacy Policy — footer links + a checkbox before checkout submission. **Omnibus Directive**: "lowest price in the last 30 days" shown on every promotion.

## DSA — Digital Services Act (EU, since 2024)
- A single point of contact (for users and authorities). Terms of Service in plain language. Dark patterns banned (fake countdown timers, hidden subscriptions, confirmshaming). Annual transparency report if the site has UGC.

## E-services law (Poland)
- Terms of Service must be free and downloadable (PDF). Marketing consent SEPARATE from Terms acceptance. Complaints: a documented procedure + `/complaints` page + response within 14 days.

## Data Processing Agreements (before going to production)
- Signed with: Vercel, Neon/Railway, Sentry, PostHog, Resend/Postmark, MailerLite, invoicing provider, Stripe, courier, Cloudflare. Store at `docs/dpa/<processor>.pdf`. Annual audit (SOC2/ISO expiry → find an alternative).

## Minimum footer / imprint
- Company name + legal form, registered address, tax/registration IDs, contact email + phone. Links: terms, privacy policy, accessibility statement, cookie settings (re-open). Year + copyright.

## KSeF — national e-invoicing (Poland; in force: 01.02.2026 largest taxpayers, 01.04.2026 everyone else)
- Every B2B invoice (customer with a NIP) MUST go through KSeF via a certified provider (Fakturownia/inFakt) — they handle signing, submission, and UPO. B2C stays a classic PDF invoice.
- Store `ksef_id` + UPO in `order.metadata`. Offline fallback: flag `ksef_pending`, hourly retry cron, alert after 48h without UPO. NEVER generate your own invoice numbers.

## GPSR — General Product Safety Regulation (in force since 13.12.2024)
- Stores selling physical products MUST show, at the product, the manufacturer's OR EU responsible person's details (name, postal address, e-mail) + safety warnings in the consumer's language. Applies to marketplaces/dropshipping too.

## AI Act — transparency (when the site uses AI)
- An AI chatbot must disclose that the user is talking to AI (never impersonate a human). Label synthetic/AI-generated content where required.

## Data retention (PL tax + GDPR)
- Invoices + order data: 5 years (Polish tax law), then anonymize PII (yearly cron). Guest/abandoned carts: 90-day TTL hard delete. Security logs: 12 months.

## Tier 2 (copy)
`Syntance/moduly` → `packages/legal-consent` (`ConsentProvider`, `CookieConsent`, terms/privacy/returns page templates).
