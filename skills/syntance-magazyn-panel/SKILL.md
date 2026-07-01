---
name: syntance-magazyn-panel
description: "Magazyn" admin panel + CMS on metadata + file storage - the @moduly/magazyn-* modules (products, orders, categories, CMS, emails, forms, returns, GA4/PostHog stats, settings), admin session, S3/R2 upload, content revalidation. Activate when building/editing an admin panel, an in-panel CMS, an email editor, upload handling, or sales statistics.
---

# "Magazyn" Admin Panel + CMS + Storage

Syntance's admin panel, mounted under `/magazyn/panel`. Don't build from scratch — compose it from `@moduly/magazyn-*` packages (→ `syntance-moduly-deploy`).

## Architecture
- Core: `@moduly/magazyn-core` (admin session, env, Medusa client, storage, audit log). Admin auth with MFA (→ `syntance-security`).
- Feature modules: `magazyn-products`, `-orders`, `-categories`, `-content` (CMS), `-emails`, `-forms`, `-returns`, `-analytics`, `-settings`.
- Panel UI/chrome: `@moduly/ui` (sidebar, panel-shell, primitives). Theme in `theme.css`.

## CMS on metadata
- Page content is stored on `Store.metadata` (parser + keys in `packages/cms`). Edit in the panel → save → revalidate (`revalidateTag`) + optional Vercel redeploy.
- Sanitize rich text BEFORE saving (protection at rest). Per-page SEO lives in `magazyn-content/seo` + `packages/seo-geo`.
- General CMS decision guide: Sanity (client edits content), Payload (dev + auth), MDX (dev-only). Here: a custom CMS on metadata for moduly-based stores.

## Transactional emails
- Block-based editor in `magazyn-emails`, sent via **Resend**. Templates: order confirmation, bank transfer instructions, contact, payment-failed. Rendering + inline HTML sanitization.

## Forms / returns
- `magazyn-forms`: forms + submission inbox + rate limiting + Zod validation. `magazyn-returns`: returns/claims + customer portal (OTP).

## Analytics
- `magazyn-analytics`: GA4 + PostHog dashboard at `/panel/statystyki` (demo or live API). Consent-aware.

## File storage
- Upload via signed URLs (S3/R2), never through your own API. Validate MIME type + magic bytes. See `cms-media-gate`, `asset-url`, `upload` in `magazyn-core/storage`.

## Tier 2 (copy)
`Syntance/moduly` → `packages/magazyn-*`, `packages/ui`, `packages/cms`, `packages/client-panel`, `apps/starter-sklep/app/magazyn`.
