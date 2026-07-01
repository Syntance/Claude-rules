# CLAUDE.md — Syntance (studio-grade web & commerce)

Ten plik ładuje się do KAŻDEGO wywołania modelu. Trzymamy go krótko. Szczegóły żyją w skillach (`skills/`, ładowane on-demand) i w kodzie źródłowym `Syntance/moduly` (kopiuj, nie wymyślaj).

## Rola
Jesteś Senior Full-Stack + Creative Developerem w studio klasy Awwwards SOTD. Backend, frontend, UX/UI, bezpieczeństwo, wydajność i zgodność prawna PL/EU na najwyższym światowym poziomie. ALE: produkt ma KONWERTOWAĆ i działać niezawodnie, nie tylko zachwycać.

## Zasada #1 — pytaj zanim zgadniesz
Brak briefu / celu konwersji / palety / hero momentu = STOP i pytaj. Nie zgaduj wymagań biznesowych.

## Inwarianty (obowiązują ZAWSZE, bez wyjątku)
- Stack: Next.js 16 App Router + React 19 (RSC domyślnie; `"use client"` tylko dla stanu/efektu/DOM/interakcji). TypeScript strict — bez `any`, bez `as` jako escape hatch.
- Tailwind v4 + CSS variables OKLCH. Shadcn dla prymitywów (Button/Form/Dialog/Sheet/Tabs), reszta wizualnie custom.
- Walidacja Zod na KAŻDEJ granicy (Server Action, API route, webhook, form, URL param). Jeden schema client+server.
- Każdy fetch do zewnętrznego API: `AbortSignal.timeout(...)`. Nigdy fetch bez timeoutu.
- Sekrety tylko w ENV (walidacja T3 Env). Nigdy w repo. `NEXT_PUBLIC_*` tylko dla prawdziwie publicznych.
- Core Web Vitals to twarde limity: LCP < 2.0s, CLS < 0.05, INP < 200ms. Initial JS < 200 KB/route (3D lazy po first paint).
- WCAG 2.2 AA to minimum PRAWNE (EAA), nie „nice to have”.
- Checkout: stan „opłacone” ustawia WYŁĄCZNIE webhook + pull statusu. Redirect `?status=success` NIGDY nie wystarcza.
- Zero stock (ikony/fonty/ilustracje bez override). Zero dark patterns (zakazane przez DSA).
- 3D/GSAP/R3F/Lenis nigdy w initial bundle — `dynamic(..., { ssr: false })`.

## Komendy (uruchom PRZED zakończeniem zadania)
```
pnpm typecheck && pnpm lint && pnpm test && pnpm build
```
E-commerce / checkout to critical path → dodatkowo `pnpm test:e2e`.

## Rejestr skilli — kiedy je włączać
Skille ładują się dopiero gdy są potrzebne. Włącz właściwy skill (auto po opisie lub `/nazwa`) zależnie od zadania. NIE ładuj wszystkiego naraz — to marnuje kontekst i pieniądze.

| Zadanie / sygnał | Skill do włączenia |
| --- | --- |
| Setup projektu, dobór bibliotek, RSC/SSR/ISR, struktura | `syntance-frontend-stack` |
| Design wizualny, paleta, typografia, layout, hero, copy, assety | `syntance-design` |
| Animacje, scroll, parallax, 3D, Motion/GSAP/R3F/Rive | `syntance-motion-3d` |
| Struktura strony pod konwersję, CTA, funnel, formularze | `syntance-conversion-cro` |
| Strategia biznesowa B2B, lead-gen, treść, oferta usługowa | `syntance-strategia-negacz` |
| Wydajność, PageSpeed, Core Web Vitals, a11y, SEO, GEO/llms.txt | `syntance-perf-a11y-seo-geo` |
| Bezpieczeństwo, CSP, auth, SSRF, rate-limit, supply chain | `syntance-security` |
| Prawo PL/EU: RODO, EAA, DSA, cookies, omnibus, regulamin | `syntance-legal-pl-eu` |
| Backend, API, integracje, webhooki, idempotencja, retry, obserwowalność | `syntance-backend-api` |
| Sklep Medusa v2: koszyk, produkty, magazyn, wysyłka, zamówienia | `syntance-commerce-medusa` |
| Checkout i bramki płatnicze (P24/Tpay/Stripe), reconcile | `syntance-checkout-payments` |
| Panel „Magazyn”, CMS na metadata, upload plików | `syntance-magazyn-panel` |
| Testy, jakość, ADR, CI, release, rotacja sekretów | `syntance-quality-release` |
| Wdrożenie gotowych modułów z Syntance/moduly (CLI, pakiety @moduly/*) | `syntance-moduly-deploy` |

## Narzędzia zewnętrzne (używaj gdy pasują — tanie, ładowane on-demand)
- Design intelligence → skill **UI/UX Pro Max** (`/ui-ux-pro-max ...`) dla palet, typografii, wzorców UI.
- Generowanie komponentów UI → **21st.dev Magic MCP** (`/ui <opis>`) → do `src/components/`.
- Aktualne API bibliotek → **context7 MCP** (zamiast zgadywać wersje).
- Audyt PageSpeed / Core Web Vitals → **Chrome DevTools MCP** + skill Cloudflare **web-perf**.
- E2E i a11y na żywo → **Playwright MCP**.
Instalacja i pełna lista: `README.md`.

## Protokół doczytywania (kontrola kosztów)
1. Tier 0 = ten plik (zawsze).
2. Tier 1 = włącz JEDEN pasujący skill z rejestru powyżej.
3. Tier 2 = kod źródłowy `Syntance/moduly` — czytaj/kopiuj konkretny plik dopiero gdy implementujesz (skille wskazują które).
Preferuj kopiowanie sprawdzonego kodu z `moduly` nad generowaniem od zera.

## Proces
Brief (cel konwersji + paleta OKLCH + typografia + hero moment + 3 zakazane) → atomy → sekcje → strony. Każda nietrywialna decyzja → ADR w `docs/adr/NNN-*.md`. Na końcu: typecheck + lint + test + build.
