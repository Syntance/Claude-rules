# Audyt Claude-rules + projekt poprawek (2026-07-07)

Repo jest portem `cursor-rules` do Claude Code (CLAUDE.md + skille, progressive
disclosure). Port jest zwięzły i dobrze skrojony pod koszt kontekstu, ale przy
kompresji wypadło kilka warstw krytycznych dla celów: ochrona przed utratą baz
danych, checkout „ponad Lumine", dyscyplina agenta, część prawa PL (KSeF/GPSR)
i płatnicze tematy ryzyka (3DS2/PCI/fraud). Poniżej projekt poprawek.

Uwaga: poprawki wspólne z cursor-rules (sprzeczności, CSP, backup, SOTD gate)
są opisane szczegółowo w `cursor-rules/docs/audyt-2026-07-poprawki.md` — tu
wskazuję, gdzie w TYM repo mają wylądować.

---

## P0 — Luki krytyczne

### P0.1 Brak całej warstwy release-ops: backup / DR / incydenty / rollback
`syntance-quality-release` kończy się na CI/CD i runbookach. Nie ma NIC o:
backupach (RPO/RTO), disaster recovery, rollbacku produkcyjnym, incident severity,
load/chaos testach, SLO, monitoringu alertowym. To wprost cel „zabezpieczenie
utraty baz danych".

Poprawka: nowy skill **`syntance-release-ops`** (osobny — czysty trigger na
„deploy/backup/incydent/monitoring", quality-release zostaje o kodzie i testach):
- Backup: klasa projektu → wymogi. Sklep/transakcyjny: **PITR** (Neon/Railway WAL)
  RPO ≤ 15 min + daily snapshot + weekly offsite. Strona: daily.
- 3-2-1 + odporność na kompromitację: kopia offsite na osobnym koncie/kluczu
  (prod credentials nie mogą kasować backupów), immutability (S3 Object Lock),
  szyfrowanie. „Nieprzetestowany backup = brak backupu": restore drill kwartalnie,
  mierzony czas = realny RTO.
- Migracje: expand → migrate → contract; zakaz DROP/RENAME w tym samym deployu
  co kod; snapshot przed każdą migracją prod.
- Rollback: Vercel promote previous (~15s), nigdy `git revert` + push w incydencie;
  kryteria rollbacku (error rate > 1%, LCP regresja, critical flow broken).
- Incident severity SEV1–4 + pierwsze 15 minut. Alert matrix (error rate, 5xx,
  p75 LCP/INP, DB pool). SLO: 99.9% uptime.
- **Dead-man's switch na crony** (Sentry Cron Monitoring / healthchecks.io):
  alert gdy backup job lub reconcile się NIE wykonał.
- Load test (k6) przed Black Friday; chaos test kwartalnie na staging.

Do `CLAUDE.md` (registry) dopisać wiersz:
`| Deploy, backup/DR, rollback, incidents, monitoring/SLO | syntance-release-ops |`

### P0.2 Brak dyscypliny agenta („koduj jak Fable 5")
Żaden tier nie mówi agentowi, czego nie wolno mu zrobić z infrastrukturą/danymi.
Poprawka: krótki blok w `CLAUDE.md` (Tier 0 — musi obowiązywać każdy call):

```
## Agent conduct (always)
- Read before you edit; match the file's conventions. Root cause > patch.
- Minimal diff: no dead code, no TODO placeholders, no drive-by refactors.
- Never guess library APIs — verify via context7 MCP.
- "Done" = quality gate actually ran and passed; report failures truthfully.
- NEVER weaken security or tests to make CI pass (no skipped tests, no `any`,
  no disabled Zod/CSP/ESLint without explicit approval).
- Destructive actions ALWAYS need explicit human approval: force-push to main,
  `migrate reset`/`db push --force-reset`/DROP/TRUNCATE on any non-local DB,
  deleting shipped migrations, `rm -rf` outside the task dir, touching prod env/secrets.
- Prod DB is read-only for the agent; every prod migration is preceded by a snapshot.
```

### P0.3 `syntance-legal-pl-eu` — braki prawne
- **KSeF** (obowiązuje: 1.02.2026 najwięksi, 1.04.2026 pozostali): faktura B2B
  (klient z NIP) przez certyfikowanego dostawcę (Fakturownia/inFakt), UPO w
  `order.metadata`, tryb offline + retry (KSeF miewa awarie), zakaz własnej
  numeracji faktur. B2C — klasyczny PDF.
- **GPSR** (od 13.12.2024): przy produkcie fizycznym dane producenta/osoby
  odpowiedzialnej w UE (nazwa, adres, e-mail) + ostrzeżenia.
- **AI Act**: chatbot AI na stronie → obowiązek poinformowania, że to AI;
  oznaczanie treści syntetycznych, gdzie wymagane.
- Retencja PL: faktury + order data 5 lat, potem anonimizacja PII (cron);
  guest/abandoned cart TTL 90 dni; logi bezpieczeństwa 12 mies.

### P0.4 `syntance-checkout-payments` — tematy ryzyka płatniczego wycięte przy porcie
Dodać zwięzłe sekcje (parytet z `medusa/payment-flow.mdc`):
- **3DS2/SCA (PSD2):** stan `RequiresAction` w maszynie stanów; Stripe
  `requires_action` → `confirmCardPayment`; timeout challenge 10 min; test kartą
  wymuszającą 3DS w staging.
- **PCI DSS SAQ A:** karta nigdy nie dotyka serwera (Elements iframe / redirect);
  NIGDY własny formularz karty; zakaz `card_number`/`cvv`/tokenów w logach —
  pre-commit `git-secrets` + Sentry scrubber.
- **Fraud:** Stripe Radar (block > 75, review 50–75), wymuszenie 3DS dla
  high-risk (kwota > 500 PLN, nowy klient, billing ≠ shipping country),
  velocity check (> 3 zamówienia z IP w 10 min → alert).
- **Chargeback:** `charge.dispute.created` → status `disputed` + alert; 7 dni
  na evidence. Refund tylko z Admin API/UI, nigdy ze storefrontu.

### P0.5 `syntance-checkout-payments` — „lepiej niż Lumine" (monitoring + testy)
- **Dead-man's switch** na cron reconcile (jak P0.1) — 5 torów domknięcia jest,
  ale żaden alert nie wykryje, że tory cicho przestały chodzić.
- **Dzienne uzgodnienie finansowe:** raport transakcje z API bramki vs zamówienia
  w DB, co do grosza; drift → alert. Łapie błędy niewidoczne dla per-koszykowego
  reconcile.
- **Success-rate per provider** z alertem przy spadku vs baseline.
- **Testy regresji anty-fałszerskiej:** `?status=success` bez płatności nie tworzy
  zamówienia (E2E); manipulacja kwotą po stronie klienta → serwer liczy z DB;
  podwójny `completeCart` (return + reconcile) → jeden order; webhook bez podpisu
  → 401 i zero zmian stanu.
- Rate-limit fail-open bez Upstash: na prod = głośny alert, nie tryb pracy.
- Express checkout (Apple/Google Pay): nie omija złotej zasady — order tworzy
  wyłącznie `completeCart` po potwierdzeniu, nigdy frontend.

---

## P1 — Ważne uzupełnienia

### P1.1 `syntance-security` — domknięcia
- CSP pełny szkielet: `default-src 'self'; base-uri 'none'; object-src 'none';
  frame-ancestors 'none'` (clickjacking), `form-action 'self' + domeny bramek`,
  `upgrade-insecure-requests` (dziś skill ma tylko script/style/HSTS/COOP).
- Supply chain 2026: pnpm **`minimumReleaseAge`** (cooldown ≥ 4–7 dni na nowe
  wersje — ochrona przed atakami npm), pin GitHub Actions po SHA.
- Passkeys (WebAuthn) jako preferowany drugi czynnik; TOTP fallback. MFA/passkey
  MUST dla publicznie dostępnego panelu admina na prod (spójnie z
  `syntance-magazyn-panel`, gdzie dziś MFA jest tylko wzmianką).

### P1.2 `syntance-commerce-medusa` — inventory races
Dwie linijki, których brakuje po kompresji: reservation = soft hold z TTL 15 min
(release po timeout, permanent decrement po `order.placed`); DB constraint
`quantity >= 0` + `SERIALIZABLE` dla createReservation — inaczej flash sale =
oversell i chargebacki.

### P1.3 `syntance-design` — SOTD self-review gate
Przed oddaniem strony samoocena wg kryteriów Awwwards (Design 40 / Usability 30 /
Creativity 20 / Content 10): jeden zapadający moment; **motion identity**
(2–3 sygnaturowe ruchy używane konsekwentnie — dopisać też w `syntance-motion-3d`);
interaction inventory (każdy element interaktywny ma hover/focus/active z motion);
typografia jako design, nie nośnik; grading fotografii; usability ścieżki
konwersji nietknięta (keyboard path + task completion).

### P1.4 `syntance-quality-release` — DoD na realnym sprzęcie
Dopisać: test na fizycznym iPhone Safari (scroll, animacje, 100dvh, widgety
bramek/InPost) przed oddaniem — emulator nie wystarcza.

---

## P2 — Spójność i higiena
- **DOMPurify:** skill security ma już wersję poprawną („avoid isomorphic-dompurify")
  — po naprawie `cursor-rules/55-security` oba repo będą spójne. Pilnować parytetu.
- **View Transitions / Turnstile-only / MFA panelu:** po zmianach w cursor-rules
  zsynchronizować brzmienie (skille już są bliżej wersji poprawnej).
- README: instrukcje instalacji OK; dopisać `syntance-release-ops` do struktury
  i tabeli „which skill for which task".
- CLAUDE.md limit ~180 linii: po dodaniu „Agent conduct" (+9 linii) i wiersza
  registry (+1) nadal w budżecie (~76 linii obecnie).

## Kolejność wdrożenia
1. P0.2 (Agent conduct w CLAUDE.md — 10 minut, działa od razu na każdy call).
2. P0.1 (`syntance-release-ops`) — backup/DR/incydenty.
3. P0.4 + P0.5 (checkout-payments) i P0.3 (legal).
4. P1.x, potem P2 (synchronizacja z cursor-rules po ich poprawkach).
