---
name: syntance-quality-release
description: Code quality, testing, ADRs, CI/CD, and release management - TS strict, ESLint, Vitest, Playwright E2E, commit conventions, Architecture Decision Records, CI pipelines, deployment (Vercel/Railway), secret rotation, runbooks. Activate when configuring tests, CI, preparing a release/deploy, making architectural decisions, or doing code review.
---

# Quality and Release

## Quality gate (before finishing any task)
```
pnpm typecheck && pnpm lint && pnpm test && pnpm build
```
Critical paths (checkout/payments/auth) → also run `pnpm test:e2e`.

## TypeScript / lint
- `strict: true`. No `any`, no `as` as an escape hatch. Shared types in `packages/types`.
- ESLint + Prettier (via turbo). No dead code, no `console.log` in production.

## Testing
- **Vitest** for unit logic (error classification, idempotency, parsers, validation).
- **Playwright** E2E for critical paths: checkout happy path, a11y baseline (axe), store/site smoke tests.
- Test integration contracts (webhooks, reconcile) against the provider's sandbox.

## ADRs (Architecture Decision Records)
- Every non-trivial decision → `docs/adr/NNN-title.md`: context, options considered, decision, consequences. A philosophy change → PR + ADR.

## CI/CD
- `pnpm install --frozen-lockfile`. Stages: install → typecheck → lint → test → build → (e2e on staging). socket.dev + `pnpm audit` (fail on critical).
- Preview deploy per PR (Vercel). Promote to production after CI is green. Document rollback steps.
- Related skills: Vercel `deployments-cicd`, `vercel-cli`.

## Secrets / release security (→ `syntance-security`)
- ENV in Doppler/Vercel, validated via T3 Env (build fails if required vars are missing). Rotate quarterly and immediately after a leak or team departure.
- GitHub secret scanning + push protection ON. `.env` in `.gitignore`.

## Runbooks
- `docs/runbook/`: data breach (72h notification, → `syntance-legal-pl-eu`), payment incident (reconcile), rollback. Include contacts (DPA, DPO, providers).

## End-to-end verification
- After deploy, run a production smoke test (incognito): the key path + confirm no PII leaks into Sentry. Use the Vercel `verification` skill or Playwright MCP.
