---
name: syntance-checkout-payments
description: Checkout i bramki płatnicze Medusa v2 (Przelewy24, Tpay, Stripe, przelew) — twardy standard utwardzony na incydentach produkcyjnych: 5 torów domknięcia płatności, kontrakt adaptera bramki, idempotencja sesji, self-healing reconcile, formularz checkoutu, security/CSP, Turnstile za flagą. Włącz przy pracy nad checkoutem, dodawaniu bramki, webhookach płatności, reconcile, formularzu zamówienia.
---

# Checkout & Payments (standard Syntance)

Nie projektuj od zera — KOPIUJ z `Syntance/moduly` (`packages/commerce`, `apps/backend`) i podmień klucze/bramkę.

## 0. Złota zasada
Stan „opłacone” ustawia WYŁĄCZNIE webhook providera + pull `getPaymentStatus`/`verify`. Redirect `?status=success` NIGDY nie wystarcza (można sfałszować). Zamówienie powstaje tylko po realnym potwierdzeniu środków (`completeCartWorkflow`).

## 1. Pięć torów domknięcia (wszystkie MUST)
1. **Return page** `/checkout/<provider>/return` — pull status → `completeCart` (polling z backoffem).
2. **Webhook** (`getWebhookActionAndData`) — weryfikacja podpisu PRZED logiką → `verify` → aktualizacja sesji. Nie woła `completeCart`.
3. **Reconcile scheduled job** (`jobs/reconcile-<provider>-payments.ts`) — co 10 min domyka sieroty. Działa tylko w worker mode shared/worker.
4. **Reconcile HTTP endpoint + cron Vercel** — siatka NIEZALEŻNA od workera (na 1 instancji „server” joby nie chodzą!).
5. **Idempotencja + fallback** — reuse pending sesji + circuit breaker → przelew tradycyjny gdy bramka pada.

## 2. Kontrakt adaptera bramki
- Moduł `apps/backend/src/modules/<provider>/` dziedziczy `AbstractPaymentProvider`.
- `initiatePayment` → `data.redirect_url` (redirect) lub `client_secret` (Stripe). Kwota TYLKO z `input.amount`.
- `authorizePayment` + `getPaymentStatus` pull-based. `getWebhookActionAndData` z weryfikacją podpisu.
- Provider id w `packages/commerce/src/payments/providers.ts`. Za flagą `FEATURE_<PROVIDER>=1`.

## 3. Idempotencja sesji (łapie duplikaty)
`init<Provider>Redirect` najpierw szuka istniejącej sesji `pending` z gotowym `redirect_url` i ją reużywa (nie rejestruje nowej transakcji). Bug: reload `/start` lub wielokrotny klik „Płać” → wiele porzuconych transakcji.

## 4. Self-healing reconcile
- Wspólny rdzeń `lib/run-<provider>-reconcile.ts` (job + endpoint). Endpoint `POST /store/custom/reconcile-<provider>` (sekret `x-order-email-secret`) → dispatch maila idempotentnie.
- Cron Vercel `/api/cron/reconcile-payments` (Bearer `CRON_SECRET`) co 15 min. `MEDUSA_WORKER_MODE=shared`.

## 5. Security
- Rate limit **Upstash** (nie in-memory). Podpis webhooka przed logiką. Kwota nigdy z URL/body klienta.
- `AbortSignal.timeout(30_000)` na każdy fetch (P24/Tpay/Stripe/Turnstile/VIES/GUS). Sanityzacja order notes bez `isomorphic-dompurify` (crash SSR). CSP: domeny bramek w `connect-src`/`frame-src`.

## 6. Formularz
- Double-submit guard: `useRef` (nie useState) + `disabled`. Walidacja `onBlur` (PL: kod `^\d{2}-\d{3}$`, NIP checksum). Slow-state po 3s.
- Zgody: osobne checkboxy regulamin + RODO, required, nigdy pre-checked. Guest checkout do końca.
- **Turnstile** za flagą `NEXT_PUBLIC_TURNSTILE_SITE_KEY`, domyślnie OFF (konwersja). Nigdy reCAPTCHA.

## 7. Stabilność / bugi z produkcji
- Circuit breaker per provider → fallback przelew. `completeCart` retry backoff tylko na 409.
- Webhook idempotency wbudowany w Medusa v2 (`processPaymentWorkflow`) — nie twórz własnej tabeli ani `/api/webhooks/*` (Medusa routuje `/hooks/payment/<id>`).

## 8. Deploy checklist
`typecheck+lint+test+e2e` zielone · ENV bramki + `UPSTASH_*` + `CRON_SECRET` + `ORDER_EMAIL_INTERNAL_SECRET` (ten sam FE/BE) + `MEDUSA_WORKER_MODE=shared` · webhook `https://<backend>/hooks/payment/<id>` · CSP z domenami bramek · cron 15 min · Sentry alerty (reconcile-recovered, webhook-sig-fail) · smoke: reload `/start` = 1 transakcja.

## Tier 2 (mapa kopiowania)
sanitize-order-notes, `medusa/checkout.ts` (findReusable*), `CheckoutForm.tsx`, `modules/<provider>`, `run-<provider>-reconcile.ts`, endpoint reconcile, subscriber `order-placed`, cron route, `next.config` CSP, `vercel.json`.
