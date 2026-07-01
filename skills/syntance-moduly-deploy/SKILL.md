---
name: syntance-moduly-deploy
description: Deploying Syntance's ready-made systems from the Syntance/moduly monorepo - the @syntance/moduly CLI (create site/store, add module), @moduly/* packages (commerce, magazyn, payments, cms, legal-consent, seo-geo, analytics, client-panel), ENV manifest, deploy checklist. Activate when starting a new site/store project, adding a module (CMS/forms/returns/payments/panel), or copying proven code instead of writing it from scratch.
---

# Deploying Modules from Syntance/moduly

Rule: don't write commerce, the admin panel, payments, CMS, or the legal layer from scratch. Start from a starter and add modules. Copy proven, production-hardened code.

## Source repo
`https://github.com/Syntance/moduly` — pnpm + Turborepo monorepo. Starters + `@moduly/*` packages + the `@syntance/moduly` CLI.

## CLI (fastest path)
```bash
# New site (CMS + SEO/GEO + forms + panel; Postgres/Drizzle)
pnpm dlx @syntance/moduly create strona --target ./my-site

# New store (Medusa + admin panel + payments; Medusa Postgres)
pnpm dlx @syntance/moduly create sklep --target ./my-store

# Add a module to an existing project
pnpm dlx @syntance/moduly add cms --target ./project
```

## Starters
- `apps/starter-strona` — CMS without commerce (Postgres). Panel at `/magazyn/panel`.
- `apps/starter-sklep` — store + panel + Medusa. Backend `apps/backend` (:9000).
- `apps/panel-demo` — panel UI prototype (mock data, screenshots for briefs).

## Package map (what to use for what)
| Need | Package |
| --- | --- |
| Cart, checkout, PLP/PDP, Meilisearch | `@moduly/commerce` |
| Payment gateways (backend modules) | `@moduly/payments` |
| Admin panel | `@moduly/magazyn-*` (`syntance-magazyn-panel`) |
| CMS on metadata | `@moduly/cms`, `magazyn-content` |
| SEO/GEO (metadata, JSON-LD, llms.txt, sitemap) | `@moduly/seo-geo` |
| Cookies/GDPR/legal pages | `@moduly/legal-consent` |
| GA4/PostHog analytics | `@moduly/analytics`, `analytics-events` |
| Customer portal (account, OTP returns) | `@moduly/client-panel` |
| DataStore (Postgres/Medusa), audit, idempotency | `@moduly/data-store` |
| Types, config | `@moduly/types`, `@moduly/config` |

## ENV manifest (fill in per project)
- Gateways behind flags (payment provider credentials, feature flags per provider).
- Anti-abuse: rate-limit store credentials (Upstash).
- Reconcile/emails: cron secret, internal email secret (same value on frontend + backend), `MEDUSA_WORKER_MODE=shared`.
- Optional: Turnstile site key + secret. Template: `apps/starter-sklep/.env.example`.

## Deployment order
1. `create strona`/`create sklep` → 2. fill `.env.local` (from `.env.example`) → 3. `pnpm install && pnpm dev` → 4. add modules (`add`) → 5. build checkout per `syntance-checkout-payments` → 6. `typecheck+lint+test+e2e` → 7. deploy (`syntance-quality-release`).
