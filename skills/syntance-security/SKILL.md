---
name: syntance-security
description: Cybersecurity - OWASP, Zod validation, XSS/CSRF, SSRF, SQL injection, auth (Better Auth/NextAuth), rate limiting, CSP/HTTP headers, SRI, secrets, supply chain, API and webhook security, monitoring. Activate when working on route.ts, Server Actions, middleware, next.config, integrations, file uploads, auth, or handling user input.
---

# Security (OWASP, defense in depth)

Zero trust: every input validated, every request authorized, every external call timed out.

## Validation
- **Zod** at EVERY boundary (Server Action, API body, URL params, webhook, form). One schema shared client+server. Never `as unknown as T` as a validation escape hatch.
- `dangerouslySetInnerHTML` only after sanitization. On SSR avoid `isomorphic-dompurify` (pulls in jsdom → crashes on Next 16/Turbopack) — use a lightweight HTML-stripper instead; also sanitize BEFORE writing to the database.
- User-supplied URLs: `https:` only, blacklist `javascript:`/`data:`/`file:`.

## XSS / CSRF
- JSX escapes by default. Server Actions have automatic CSRF protection — NEVER disable it.
- API mutations: origin check in middleware. Cookies: `httpOnly` + `secure` + `sameSite`, `__Host-` prefix for session cookies.

## SSRF (critical)
- Fetching a user-supplied URL: resolve DNS and validate the **RESOLVED IP** (not the hostname). Guard against DNS rebinding.
- Blacklist: `127/8`, `10/8`, `172.16/12`, `192.168/16`, `169.254/16` (cloud metadata!), `::1`, `fc00::/7`.
- `AbortSignal.timeout(5_000)`, `redirect:'error'` or re-validate the redirect target.

## SQL / auth
- Use an ORM (Prisma/Drizzle) with prepared statements. Raw SQL only via parameterized queries. Medusa: QueryService + filters.
- Auth: **Better Auth** or **NextAuth v5**, never a custom JWT implementation. Argon2id, MFA for admin accounts. Server-side sessions, 24h rotation.
- Login rate limit: 5/15min per IP+email → CAPTCHA (hCaptcha/Turnstile, never reCAPTCHA).

## Rate limiting
- **Upstash Ratelimit** in middleware (never in-memory — lost on restart/autoscale). 60/min public, 30/min mutations. 429 + `Retry-After`.

## HTTP headers (single source: next.config/middleware)
- **CSP**: `script-src` nonce + `'strict-dynamic'` (NEVER `'unsafe-inline'` for scripts). `style-src` must allow `'unsafe-inline'`/hashes (Motion/GSAP). Start with Report-Only → then enforce.
- HSTS `max-age=63072000; includeSubDomains; preload`. `X-Content-Type-Options: nosniff`. `Referrer-Policy: strict-origin-when-cross-origin`. Restrictive `Permissions-Policy`. COOP `same-origin`. Trusted Types for script.
- **SRI** on every external `<script src=cdn>` (`integrity` + `crossorigin`).

## Secrets / supply chain
- Secrets only in ENV (Doppler/Vercel), never in the repo. Use `@t3-oss/env-nextjs`. Rotate quarterly and immediately after any leak.
- `pnpm install --frozen-lockfile` in CI. socket.dev PR gate, `pnpm audit` failing on critical, Renovate. `ignore-scripts=true` in `.npmrc`.
- GitHub secret scanning + push protection + Dependabot ON.

## API / webhooks
- Webhook signatures: HMAC-SHA256, `crypto.timingSafeEqual` (NEVER `===`). Verify the signature BEFORE any business logic.
- Idempotency key (client-generated UUID, 24h TTL). API versioning `/api/v1/*`.

## File uploads
- Signed URLs (S3/R2), never upload through your own API. Validate MIME type + re-validate magic bytes (`file-type`). User-uploaded SVG is forbidden.

## Monitoring
- Sentry: CSP violations + PII scrubbing in `beforeSend` (email/phone/address/ip). Alert on brute force, scanners, CSP spikes. `/.well-known/security.txt`.
