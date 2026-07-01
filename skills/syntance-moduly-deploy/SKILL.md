---
name: syntance-moduly-deploy
description: Wdrażanie gotowych systemów Syntance z monorepo Syntance/moduly — CLI @syntance/moduly (create strona/sklep, add moduł), pakiety @moduly/* (commerce, magazyn, payments, cms, legal-consent, seo-geo, analytics, client-panel), manifest ENV, checklist deploy. Włącz gdy startujesz nowy projekt strony/sklepu, dokładasz moduł (CMS/forms/returns/payments/panel) lub kopiujesz sprawdzony kod zamiast pisać od zera.
---

# Wdrażanie modułów z Syntance/moduly

Zasada: NIE pisz od zera commerce, panelu, płatności, CMS ani warstwy prawnej. Startuj ze startera i dokładaj moduły. Kopiuj sprawdzony kod (utwardzony na produkcji).

## Repo źródłowe
`https://github.com/Syntance/moduly` — monorepo pnpm + Turborepo. Startery + pakiety `@moduly/*` + CLI `@syntance/moduly`.

## CLI (najszybsza ścieżka)
```bash
# Nowa strona (CMS + SEO/GEO + formularze + panel; Postgres/Drizzle)
pnpm dlx @syntance/moduly create strona --target ./moja-strona

# Nowy sklep (Medusa + magazyn + płatności; Medusa Postgres)
pnpm dlx @syntance/moduly create sklep --target ./moj-sklep

# Dołóż moduł do istniejącego projektu
pnpm dlx @syntance/moduly add cms --target ./projekt
```

## Startery
- `apps/starter-strona` — CMS bez commerce (Postgres). Panel `/magazyn/panel`.
- `apps/starter-sklep` — sklep + panel + Medusa. Backend `apps/backend` (:9000).
- `apps/panel-demo` — prototyp UI panelu (mock, screenshoty do briefów).

## Mapa pakietów (co skąd)
| Potrzeba | Pakiet |
| --- | --- |
| Koszyk, checkout, PLP/PDP, Meilisearch | `@moduly/commerce` |
| Płatności P24/Stripe/Tpay/przelew | `@moduly/payments` (+ moduły backend) |
| Panel admina | `@moduly/magazyn-*` (`syntance-magazyn-panel`) |
| CMS na metadata | `@moduly/cms`, `magazyn-content` |
| SEO/GEO (metadata, JSON-LD, llms.txt, sitemap) | `@moduly/seo-geo` |
| Cookies/RODO/strony prawne | `@moduly/legal-consent` |
| Analityka GA4/PostHog | `@moduly/analytics`, `analytics-events` |
| Panel klienta (konto, zwroty OTP) | `@moduly/client-panel` |
| DataStore (Postgres/Medusa), audit, idempotency | `@moduly/data-store` |
| Typy, config | `@moduly/types`, `@moduly/config` |

## Manifest ENV (uzupełnij per projekt)
- Bramki za flagą: `FEATURE_PRZELEWY24=1`, `P24_*` / `STRIPE_*` / `TPAY_*`.
- Anti-abuse: `UPSTASH_REDIS_REST_URL/TOKEN`.
- Reconcile/maile: `CRON_SECRET`, `ORDER_EMAIL_INTERNAL_SECRET` (ten sam FE+BE), `MEDUSA_WORKER_MODE=shared`.
- Opcjonalnie: `NEXT_PUBLIC_TURNSTILE_SITE_KEY`, `TURNSTILE_SECRET_KEY`. Wzór: `apps/starter-sklep/.env.example`.

## Kolejność wdrożenia
1. `create strona`/`create sklep` → 2. uzupełnij `.env.local` (wzór z `.env.example`) → 3. `pnpm install && pnpm dev` → 4. dołóż moduły (`add`) → 5. checkout wg `syntance-checkout-payments` → 6. `typecheck+lint+test+e2e` → 7. deploy (`syntance-quality-release`).

## Cursor rules (jeśli projekt używa Cursora)
```bash
pnpm dlx degit Syntance/cursor-rules/fundament .cursor/rules
pnpm dlx degit Syntance/cursor-rules/medusa   .cursor/rules
pnpm dlx degit Syntance/cursor-rules/magazyn  .cursor/rules
```
