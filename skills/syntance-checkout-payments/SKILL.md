---
name: syntance-checkout-payments
description: Checkout and payment gateways on Medusa v2 (Przelewy24, Tpay, Stripe, bank transfer) - the hard standard hardened by production incidents - five payment-closing paths, payment adapter contract, session idempotency, self-healing reconcile, checkout form, security/CSP, Turnstile behind a flag. Activate when working on checkout, adding a payment gateway, payment webhooks, reconcile jobs, or the order form.
---

# Checkout & Payments (Syntance Standard)

Don't design from scratch — COPY from `Syntance/moduly` (`packages/commerce`, `apps/backend`) and swap in the project's own keys/gateway.

## 0. Golden rule
The "paid" state is set EXCLUSIVELY by the provider's webhook + a status pull (`getPaymentStatus`/`verify`). A `?status=success` redirect is NEVER sufficient on its own (it can be forged). An order is only created after real fund confirmation (`completeCartWorkflow`).

## 1. Five payment-closing paths (all MUST)
1. **Return page** `/checkout/<provider>/return` — pull status → `completeCart` (polling with backoff).
2. **Webhook** (`getWebhookActionAndData`) — verify the signature BEFORE any logic → `verify` → update the payment session. Does not call `completeCart`.
3. **Reconcile scheduled job** (`jobs/reconcile-<provider>-payments.ts`) — runs every 10 min to close orphaned carts. Only runs in shared/worker mode.
4. **Reconcile HTTP endpoint + Vercel cron** — an independent safety net (on a single instance in "server" mode, scheduled jobs don't run at all!).
5. **Idempotency + fallback** — reuse a pending session + circuit breaker → fall back to manual bank transfer if the gateway is down.

## 2. Payment adapter contract
- Module `apps/backend/src/modules/<provider>/` extends `AbstractPaymentProvider`.
- `initiatePayment` → returns `data.redirect_url` (redirect flow) or `client_secret` (Stripe). Amount ONLY from `input.amount`.
- `authorizePayment` + `getPaymentStatus` are pull-based. `getWebhookActionAndData` verifies the signature.
- Provider id registered in one place: `packages/commerce/src/payments/providers.ts`. Gated by `FEATURE_<PROVIDER>=1`.

## 3. Session idempotency (catches duplicates)
`init<Provider>Redirect` must first look for an existing `pending` session with a ready `redirect_url` and reuse it (never register a new transaction). Bug this prevents: reloading `/start` or clicking "Pay" multiple times → multiple abandoned transactions.

## 4. Self-healing reconcile
- Shared core `lib/run-<provider>-reconcile.ts` (used by both the job and the endpoint). Endpoint `POST /store/custom/reconcile-<provider>` (secret header) → dispatches emails idempotently.
- Vercel cron `/api/cron/reconcile-payments` (Bearer secret) every 15 min. `MEDUSA_WORKER_MODE=shared`.

## 5. Security
- Rate limit via **Upstash** (never in-memory). Verify the webhook signature before any logic. Amount never taken from client URL/body.
- `AbortSignal.timeout(30_000)` on every external fetch (gateway, CAPTCHA, tax/registry lookups). Sanitize order notes without `isomorphic-dompurify` (SSR crash risk). CSP `connect-src`/`frame-src` includes gateway domains.

## 6. Form
- Double-submit guard: `useRef` (not useState) + `disabled`. Validation `onBlur`. Slow-state message after 3s.
- Consents: separate checkboxes for terms + privacy, required, never pre-checked. Guest checkout all the way through.
- **Turnstile** behind a flag (`NEXT_PUBLIC_TURNSTILE_SITE_KEY`), OFF by default (conversion). Never reCAPTCHA.

## 7. Stability / production bugs fixed by this standard
- Circuit breaker per provider → fallback to bank transfer. `completeCart` retries with backoff only on 409.
- Webhook idempotency is built into Medusa v2 (`processPaymentWorkflow`) — don't build your own table or a custom `/api/webhooks/*` route (Medusa routes `/hooks/payment/<id>`).

## 8. Deploy checklist
`typecheck+lint+test+e2e` green · gateway ENV vars + rate-limit + cron secrets set · `MEDUSA_WORKER_MODE=shared` · webhook URL registered with the provider · CSP includes gateway domains · cron active (15 min) · Sentry alerts (reconcile-recovered, webhook-sig-fail) · smoke test: reloading `/start` = 1 transaction.

## Tier 2 (copy map)
sanitize-order-notes, `medusa/checkout.ts` (findReusable*), `CheckoutForm.tsx`, `modules/<provider>`, `run-<provider>-reconcile.ts`, reconcile endpoint, `order-placed` subscriber, cron route, `next.config` CSP, `vercel.json`.
