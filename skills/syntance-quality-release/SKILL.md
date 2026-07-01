---
name: syntance-quality-release
description: Jakość kodu, testy, ADR, CI/CD i release — TS strict, ESLint, Vitest, Playwright E2E, konwencje commitów, Architecture Decision Records, pipeline CI, deploy (Vercel/Railway), rotacja sekretów, runbooki. Włącz przy konfiguracji testów, CI, przygotowaniu do release/deploy, decyzjach architektonicznych, code review.
---

# Jakość i release

## Bramka jakości (przed każdym końcem zadania)
```
pnpm typecheck && pnpm lint && pnpm test && pnpm build
```
Critical path (checkout/płatności/auth) → dodatkowo `pnpm test:e2e`.

## TypeScript / lint
- `strict: true`. Bez `any`, bez `as` jako escape. Typy współdzielone w `packages/types`.
- ESLint + Prettier (turbo). Brak martwego kodu, brak `console.log` w prod.

## Testy
- **Vitest** unit dla logiki (klasyfikacja błędów, idempotencja, parsery, walidacja).
- **Playwright** E2E dla ścieżek krytycznych: happy path checkout, a11y baseline (axe), smoke sklep/strona.
- Testuj kontrakty integracji (webhook, reconcile) na sandboxie providera.

## ADR (Architecture Decision Records)
- Każda nietrywialna decyzja → `docs/adr/NNN-tytul.md`: kontekst, opcje, decyzja, konsekwencje. Zmiana filozofii → PR + ADR.

## CI/CD
- `pnpm install --frozen-lockfile`. Etapy: install → typecheck → lint → test → build → (e2e na staging). socket.dev + `pnpm audit` (fail on critical).
- Preview deploy per PR (Vercel). Promocja do prod po zielonym CI. Rollback udokumentowany.
- Skille CI/deploy: Vercel `deployments-cicd`, `vercel-cli`.

## Sekrety / bezpieczeństwo release (→ `syntance-security`)
- ENV w Doppler/Vercel, walidacja T3 Env (build fails gdy brak required). Rotacja kwartalna + natychmiast przy leaku/odejściu.
- GitHub secret scanning + push protection ON. `.env` w `.gitignore`.

## Runbooki
- `docs/runbook/`: data-breach (72h, → `syntance-legal-pl-eu`), incydent płatności (reconcile), rollback. Kontakty (UODO, DPO, providerzy).

## Weryfikacja end-to-end
- Po deployu smoke prod (incognito): kluczowa ścieżka + brak PII w Sentry. Użyj skilla Vercel `verification` / Playwright MCP.
