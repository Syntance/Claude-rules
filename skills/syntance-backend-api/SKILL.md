---
name: syntance-backend-api
description: Niezawodność backendu, API i integracji — projektowanie API/route handlers, obsługa błędów, timeouty, retry z backoffem, idempotencja, webhooki, circuit breaker, self-healing/reconcile, kolejki, obserwowalność (Sentry/logi), kontrakty z zewnętrznymi usługami. Włącz przy tworzeniu API, integracji z zewnętrznym serwisem, webhookach, zadaniach cron, obsłudze płatności/mail/wysyłki.
---

# Backend, API i niezawodność integracji

Zasada: integracja z zewnętrznym systemem ZAWSZE potrafi zawieść. Projektuj pod porażkę sieci, opóźnienia i duplikaty.

## API / route handlers
- Walidacja Zod na wejściu, typowany wynik. Wersjonowanie `/api/v1/*`. Spójny format błędu (`{ error, code }`).
- Zwracaj właściwe kody: 400 walidacja, 401/403 auth, 409 konflikt idempotencji, 429 rate limit, 5xx serwer.
- Autoryzacja per request. Rate limit (Upstash) na endpointach wrażliwych. (→ `syntance-security`).

## Wywołania zewnętrzne (MUST)
- `AbortSignal.timeout(...)` na KAŻDYM fetch (nigdy ręczny setTimeout, nigdy bez limitu).
- Retry z **exponential backoff** TYLKO dla błędów przejściowych (sieć, 429, 503, 409 idempotency). Nie retry'uj 4xx logicznych ani zakleszczonego 500.
- Klasyfikuj błędy: transient vs permanent. Loguj z korelacją (request id).

## Idempotencja
- Operacje mutujące z zewnętrznym skutkiem (płatność, mail, zamówienie): idempotency key + zapis stanu. Duplikat → zwróć oryginalny wynik, nie wykonuj ponownie.
- Reuse istniejącej „pending” sesji zamiast tworzyć nową (wzorzec z checkoutu → `syntance-checkout-payments`).

## Webhooki (przyjmowanie)
- Weryfikuj podpis (HMAC, `timingSafeEqual`) PRZED jakąkolwiek logiką. Szybkie 2xx, ciężką pracę do kolejki/joba.
- Zakładaj duplikaty i zmianę kolejności zdarzeń → dedup po event id / stanie. W Medusa v2 dedup jest wbudowany — nie twórz własnej tabeli webhook_events.

## Self-healing / reconcile
- Dla krytycznych flow (płatności, sync) NIE polegaj na jednym torze. Zapewnij niezależną siatkę: scheduled job + HTTP endpoint + cron zewnętrzny (Vercel).
- Uwaga infra: na 1 instancji w trybie „server” Medusa scheduled jobs i subscribery NIE działają → potrzebny cron/endpoint. Domyślnie `MEDUSA_WORKER_MODE=shared`.

## Obserwowalność
- Sentry z PII scrubbing w `beforeSend`. Distinct eventy dla „reconcile odzyskał” i „webhook signature fail” (nie giną w szumie).
- Alerty na anomalie (error rate, brak zdarzeń które powinny być). Health endpoint.

## Tier 2 (kopiuj)
`Syntance/moduly` → `apps/backend/src/lib` (rate-limit, reconcile, sentry), `apps/backend/src/modules`, `packages/data-store` (idempotency, audit).
