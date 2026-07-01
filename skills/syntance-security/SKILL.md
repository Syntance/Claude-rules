---
name: syntance-security
description: Cyberbezpieczeństwo — OWASP, walidacja Zod, XSS/CSRF, SSRF, SQL injection, auth (Better Auth/NextAuth), rate limiting, CSP/nagłówki HTTP, SRI, sekrety, supply chain, bezpieczeństwo API i webhooków, monitoring. Włącz przy route.ts, Server Actions, middleware, next.config, integracjach, uploadzie plików, auth, obsłudze inputu użytkownika.
---

# Bezpieczeństwo (OWASP, defense in depth)

Zero trust: każdy input walidowany, każdy request autoryzowany, każdy external call timeoutowany.

## Walidacja
- **Zod** na KAŻDEJ granicy (Server Action, API body, URL params, webhook, form). Jeden schema client+server. Nigdy `as unknown as T` jako escape.
- `dangerouslySetInnerHTML` tylko po sanityzacji. Na SSR unikaj `isomorphic-dompurify` (jsdom → crash na Next16/Turbopack) — użyj lekkiego strip-HTML; sanityzuj też PRZED zapisem do DB.
- URL od usera: `https:` only, blacklist `javascript:`/`data:`/`file:`.

## XSS / CSRF
- JSX escape'uje domyślnie. Server Actions: ochrona CSRF automatyczna — NIGDY nie wyłączaj.
- API mutacje: origin check w middleware. Cookies: `httpOnly` + `secure` + `sameSite`, prefix `__Host-` dla sesji.

## SSRF (krytyczne)
- Fetch z URL usera: zresolwuj DNS i waliduj **ROZWIĄZANY IP** (nie host). Guard na DNS rebinding.
- Blacklist: `127/8`, `10/8`, `172.16/12`, `192.168/16`, `169.254/16` (metadata!), `::1`, `fc00::/7`.
- `AbortSignal.timeout(5_000)`, `redirect:'error'` lub re-walidacja celu redirectu.

## SQL / auth
- ORM (Prisma/Drizzle), prepared statements. Raw SQL tylko parametryzowany. Medusa: QueryService + filters.
- Auth: **Better Auth** lub **NextAuth v5**, nigdy custom JWT. Argon2id, MFA dla admina. Sesja server-side, rotacja 24h.
- Login rate limit: 5/15min per IP+email → CAPTCHA (hCaptcha/Turnstile, nie reCAPTCHA).

## Rate limiting
- **Upstash Ratelimit** w middleware (nie in-memory — ginie przy restart/autoscale). Public 60/min, mutacje 30/min. 429 + `Retry-After`.

## HTTP headers (single source: next.config/middleware)
- **CSP**: `script-src` nonce + `'strict-dynamic'` (NIGDY `'unsafe-inline'` dla skryptów). `style-src` musi mieć `'unsafe-inline'`/hashe (Motion/GSAP). Start Report-Only → enforcement.
- HSTS `max-age=63072000; includeSubDomains; preload`. `X-Content-Type-Options: nosniff`. `Referrer-Policy: strict-origin-when-cross-origin`. `Permissions-Policy` restrykcyjne. COOP `same-origin`. Trusted Types dla script.
- **SRI** na każdym zewnętrznym `<script src=cdn>` (`integrity` + `crossorigin`).

## Sekrety / supply chain
- Sekrety tylko w ENV (Doppler/Vercel), nigdy w repo. `@t3-oss/env-nextjs`. Rotacja kwartalna + przy leaku.
- `pnpm install --frozen-lockfile` w CI. socket.dev PR gate, `pnpm audit` fail on critical, Renovate. `ignore-scripts=true` w `.npmrc`.
- GitHub secret scanning + push protection + Dependabot ON.

## API / webhooki
- Webhook signatures: HMAC-SHA256, `crypto.timingSafeEqual` (NIGDY `===`). Weryfikuj podpis PRZED logiką.
- Idempotency key (UUID klienta, TTL 24h). API versioning `/api/v1/*`.

## Upload plików
- Signed URLs (S3/R2), nigdy upload do własnego API. Walidacja MIME + re-validate magic bytes (`file-type`). SVG od usera zakazany.

## Monitoring
- Sentry: CSP violations + PII scrubbing w `beforeSend` (email/phone/address/ip). Alerty: brute force, scanner, CSP spike. `/.well-known/security.txt`.
