---
name: syntance-magazyn-panel
description: Panel administracyjny „Magazyn” + CMS na metadata + storage plików — moduły @moduly/magazyn-* (produkty, zamówienia, kategorie, CMS, maile, formularze, zwroty, statystyki GA4/PostHog, ustawienia), sesja admina, upload do S3/R2, rewalidacja treści. Włącz przy budowie/edycji panelu admina, CMS w panelu, edytorze maili, obsłudze uploadu, statystykach sprzedaży.
---

# Panel „Magazyn” + CMS + storage

Panel admina Syntance montowany pod `/magazyn/panel`. Nie buduj od zera — składaj z pakietów `@moduly/magazyn-*` (→ `syntance-moduly-deploy`).

## Architektura
- Rdzeń: `@moduly/magazyn-core` (sesja admina, env, klient Medusa, storage, audit log). Auth admina z MFA (→ `syntance-security`).
- Moduły funkcjonalne: `magazyn-products`, `-orders`, `-categories`, `-content` (CMS), `-emails`, `-forms`, `-returns`, `-analytics`, `-settings`.
- UI/chrome panelu: `@moduly/ui` (sidebar, panel-shell, prymitywy). Motyw w `theme.css`.

## CMS na metadata
- Treść stron trzymana na `Store.metadata` (parser + klucze w `packages/cms`). Edycja w panelu → zapis → rewalidacja (`revalidateTag`) + opcjonalny redeploy Vercel.
- Sanityzuj rich text PRZED zapisem (protection at rest). SEO per strona w `magazyn-content/seo` + `packages/seo-geo`.
- CMS decyzja ogólna: Sanity (klient edytuje), Payload (dev + auth), MDX (dev-only). Tu: własny CMS na metadata dla sklepów moduly.

## Maile transakcyjne
- Edytor blokowy w `magazyn-emails`, wysyłka przez **Resend**. Szablony: zamówienie, przelew, kontakt, payment-failed. Render + sanityzacja inline HTML.

## Formularze / zwroty
- `magazyn-forms`: formularze + skrzynka zgłoszeń + rate limit + walidacja Zod. `magazyn-returns`: zwroty/reklamacje + panel klienta (OTP).

## Statystyki
- `magazyn-analytics`: dashboard GA4 + PostHog w `/panel/statystyki` (demo lub live API). Consent-aware.

## Storage plików
- Upload przez signed URLs (S3/R2), nie do własnego API. Walidacja MIME + magic bytes. `cms-media-gate`, `asset-url`, `upload` w `magazyn-core/storage`.

## Tier 2 (kopiuj)
`Syntance/moduly` → `packages/magazyn-*`, `packages/ui`, `packages/cms`, `packages/client-panel`, `apps/starter-sklep/app/magazyn`.
