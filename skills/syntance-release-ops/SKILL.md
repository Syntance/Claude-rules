---
name: syntance-release-ops
description: Release operations - deploy, rollback, backup/DR (PITR, 3-2-1, restore drills), database migration safety (expand-contract), incident response (SEV1-4), monitoring/alerting, SLO, load and chaos testing, dead-man's switch on crons. Activate when deploying, configuring backups, planning disaster recovery, handling an incident, setting up monitoring/alerts, or preparing for high-traffic events.
---

# Release Ops (deploy, backup/DR, incidents)

Beautiful and fast MUST also survive production. Data loss is the one unrecoverable failure — design backups first.

## Backup (RPO per project class)
- **Store / transactional app**: PITR/WAL (Neon point-in-time restore, Railway wal-g), RPO ≤ 15 min + daily snapshot + weekly full export offsite. A daily backup alone = up to 24h of lost orders — not acceptable.
- **Content site**: daily automated + 30-day retention + weekly export is enough.
- **3-2-1 + compromise-resistant**: offsite copy on a SEPARATE account/key — prod credentials must NOT be able to delete backups. Versioning + immutability (S3 Object Lock) for the weekly copy. Encrypted at rest.
- **Restore drill quarterly** (rebuild staging from the backup). Measure the restore time and record it in the runbook — that is the real RTO. "An untested backup is no backup."

## DB migrations (zero data loss)
- **Expand → migrate → contract**: add the new column/table first, switch the code, drop the old one in a SEPARATE deploy.
- NEVER DROP/RENAME a column in the same deploy as the code that uses it.
- Snapshot BEFORE every prod migration. Migration rollback described in the PR. Migrations run in CI/CD, never by hand from a laptop.

## Deploy / rollback
- Preview (per PR) → Staging (merge to main) → Prod (tag/manual promote). Zero downtime. Risky deploys Mon–Thu 10:00–15:00.
- Rollback via platform promote-previous (~15s on Vercel), NEVER `git revert` + push during an incident. Triggers: error rate > 1% for 5 min, LCP regression > 500ms p75, any 5xx spike, critical flow broken (checkout/auth).
- Progressive delivery: risky features behind flags, rollout 1% → 10% → 50% → 100%. NEVER instant 100% for DB schema, payment/checkout/auth changes. Skew protection on.

## Incidents (first 15 minutes)
- SEV1 (site/checkout down, data breach): respond in 15 min, rollback IMMEDIATELY. SEV2 (feature broken, perf regression > 30%): respond 1h, fix in 24h. SEV3/4: planned.
- Confirm scope in Sentry/analytics (1 user / 10% / 100%) → > 10% = rollback now; < 10% = feature flag off. Post-mortem within 24h (timeline, root cause, prevention). Data breach → 72h notification (→ `syntance-legal-pl-eu`).

## Monitoring / alerting
- Alert matrix: error rate > 1% → instant; 5xx spike → instant; p75 LCP > 2.5s / INP > 300ms → daily summary; DB pool > 80% → email; deploy failure → instant.
- **Dead-man's switch on crons**: Sentry Cron Monitoring / healthchecks.io on backup jobs and payment reconcile — alert when the job DIDN'T run (crons fail silently; alert on absence, not just on errors).
- Uptime ping every minute. Health endpoints: `/api/health` + `/api/health/deep` (DB/Redis ping), never secrets in responses.
- SLO: 99.9% uptime, LCP p75 < 2.5s for 95% of the time, error rate < 0.5%. Contractual SLA only for premium.

## Load / chaos
- k6 before major releases / Black Friday: 100 concurrent home, 50 PDP, 20 checkout (test mode). Pass: p95 < 2s, error rate < 0.1%, zero 5xx.
- Chaos quarterly on staging: kill the DB → graceful error + recovery; throttle APIs → loaders work; break the gateway URL → checkout shows "temporarily unavailable" + bank-transfer fallback.

## Runbooks
- `docs/runbook/*.md` per incident type: payment-down, db-connection, data-breach, cdn-404, certificate-expired. Each: symptoms / diagnosis / fix / prevention + contacts.

## Tier 2 (copy)
`Syntance/moduly` → `apps/backend/src/lib` (health, sentry), `vercel.json` (crons), runbook templates.
