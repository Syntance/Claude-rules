---
name: syntance-legal-pl-eu
description: Zgodność prawna PL/EU — EAA (dostępność), RODO/GDPR (prawa podmiotu, breach 72h), DSA, ustawa o prawach konsumenta, cookies (art. 173 PT), omnibus, ustawa o świadczeniu usług, DPA z procesorami, impressum/stopka. Włącz przy stopce, checkoutcie, formularzach zgód, cookies, stronach prawnych, deklaracji dostępności.
---

# Prawo PL/EU (compliance)

## EAA — European Accessibility Act (obowiązkowy)
- Scope: wszystkie B2C digital products w UE. Wymóg: **WCAG 2.2 AA** (→ `syntance-perf-a11y-seo-geo`). Kara PL ~80 000 PLN + blokada sprzedaży.
- Strona `/deklaracja-dostepnosci`: status zgodności, data przeglądu, metoda oceny, kontakt koordynatora, link UODO/RPO.
- Przycisk „Zgłoś problem z dostępnością” w stopce. Audit roczny (axe + Wave + manual SR), fix critical w 30 dni.

## RODO / GDPR — prawa podmiotu (DSR)
- Dostęp (15), sprostowanie (16), usunięcie (17), ograniczenie (18), przenoszenie (20), sprzeciw (21).
- Implementacja: `/moje-konto/prywatnosc` z CTA download/delete/restrict/export + webhook do wszystkich procesorów (Sentry, PostHog, Resend, MailerLite).
- Retention: faktury 5 lat (Ordynacja art. 70), logi bezpieczeństwa 12 mies. (uzasadniony interes).
- **Breach 72h**: ocena ryzyka 24h → notyfikacja UODO 72h → komunikacja userów gdy wysokie ryzyko. Runbook `docs/runbook/data-breach.md`.

## Cookies (Prawo telekom. art. 173)
- Zgoda **opt-in** (nie pre-checked). Trzy równoważne przyciski **Akceptuj / Odrzuć / Dostosuj**. Nigdy „X” jako dismiss.
- Banner nie blokuje scroll. Consent gate: GA/Meta/PostHog/Hotjar ładują się TYLKO po zgodzie (+ Partytown, Consent Mode v2).
- Narzędzie: **Klaro** (OSS) lub Cookiebot. Kategorie: Necessary/Analytics/Marketing/Preferences. Retention explicite.

## Ustawa o prawach konsumenta (B2C)
- Prawo odstąpienia 14 dni (formularz + email po wysyłce). Informacja o przedsiębiorcy (nazwa, adres, NIP, REGON, KRS, kontakt).
- Regulamin + Polityka prywatności — linki w stopce + checkbox przed submit. **Omnibus**: „najniższa cena z 30 dni” przy promocji.

## DSA (od 2024)
- Punkt kontaktowy (user + władze). Regulamin prostym językiem. Zakaz dark patterns (fake timery, ukryte subskrypcje, confirmshaming). Transparency report gdy UGC.

## Ustawa o świadczeniu usług drogą elektroniczną
- Regulamin nieodpłatnie + do pobrania (PDF). Zgoda marketingowa OSOBNO od regulaminu. Reklamacje: procedura + `/reklamacje` + odpowiedź w 14 dni.

## DPA z procesorami (przed produkcją)
- Podpisane z: Vercel, Neon/Railway, Sentry, PostHog, Resend/Postmark, MailerLite, Fakturownia, Stripe, InPost, Cloudflare. Storage `docs/dpa/<processor>.pdf`. Audit roczny (SOC2/ISO wygasa → alternatywa).

## Impressum / stopka minimum
- Firma + forma prawna, adres, NIP/REGON/KRS, email + telefon. Linki: regulamin, polityka prywatności, deklaracja dostępności, cookies (re-open). Rok + prawa autorskie.

## Tier 2 (kopiuj)
`Syntance/moduly` → `packages/legal-consent` (`ConsentProvider`, `CookieConsent`, szablony regulamin/polityka/zwroty).
