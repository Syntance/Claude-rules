---
name: syntance-backend-api
description: Backend, API, and integration reliability - API/route handler design, error handling, timeouts, retry with backoff, idempotency, webhooks, circuit breakers, self-healing/reconcile jobs, queues, observability (Sentry/logs), external service contracts. Activate when building an API, integrating an external service, handling webhooks, cron jobs, or payment/email/shipping logic.
---

# Backend, API, and Integration Reliability

Rule: any integration with an external system CAN fail. Design for network failures, latency, and duplicates.

## API / route handlers
- Validate input with Zod, return a typed result. Version APIs `/api/v1/*`. Consistent error shape (`{ error, code }`).
- Return the right status codes: 400 validation, 401/403 auth, 409 idempotency conflict, 429 rate limit, 5xx server error.
- Authorize every request. Rate limit (Upstash) on sensitive endpoints. (→ `syntance-security`).

## External calls (MUST)
- `AbortSignal.timeout(...)` on EVERY fetch (never a manual setTimeout, never unbounded).
- Retry with **exponential backoff** ONLY for transient errors (network, 429, 503, 409 idempotency). Never retry logical 4xx errors or a permanently stuck 500.
- Classify errors as transient vs permanent. Log with correlation (request id).

## Idempotency
- Mutating operations with external side effects (payment, email, order): idempotency key + persisted state. Duplicate request → return the original result, don't re-execute.
- Reuse an existing "pending" session instead of creating a new one (pattern from checkout → `syntance-checkout-payments`).

## Webhooks (receiving)
- Verify the signature (HMAC, `timingSafeEqual`) BEFORE any business logic. Return a fast 2xx, push heavy work to a queue/job.
- Assume duplicates and out-of-order delivery → dedupe by event id/state. Medusa v2 has built-in dedup — don't build your own `webhook_events` table.

## Self-healing / reconcile
- For critical flows (payments, sync), never rely on a single path. Build an independent safety net: scheduled job + HTTP endpoint + external cron (Vercel).
- Infra caveat: on a single Medusa instance in "server" worker mode, scheduled jobs and subscribers do NOT run → you need a cron/endpoint. Default to `MEDUSA_WORKER_MODE=shared`.

## Observability
- Sentry with PII scrubbing in `beforeSend`. Distinct events for "reconcile recovered an order" and "webhook signature failed" (don't let them get lost in the noise).
- Alert on anomalies (error rate, missing expected events). Health endpoint.

## Tier 2 (copy)
`Syntance/moduly` → `apps/backend/src/lib` (rate-limit, reconcile, sentry), `apps/backend/src/modules`, `packages/data-store` (idempotency, audit).
