---
name: syntance-perf-a11y-seo-geo
description: Wydajność (Core Web Vitals, PageSpeed Insights), dostępność (WCAG 2.2 AA), SEO techniczne i GEO (Generative Engine Optimization — llms.txt, JSON-LD, treść answer-ready pod AI). Włącz przy optymalizacji szybkości, audycie Lighthouse/PageSpeed, pracy nad a11y, metadanymi, sitemap/robots, widocznością w wyszukiwarkach i silnikach AI.
---

# Wydajność, a11y, SEO, GEO

## Core Web Vitals (twarde limity)
- LCP < 2.0s (cel 1.5s, obejmuje preloader). CLS < 0.05. INP < 200ms (cel 100ms). FCP < 1.5s. TTFB < 500ms.
- Initial JS < 200 KB gz/route (3D lazy po first paint). Sprawdź `@next/bundle-analyzer`.
- LCP image ≤ 120 KB AVIF, next/image z `priority` + `fetchPriority="high"` + `sizes`. Fonts: next/font `display:swap`.
- Animuj tylko `transform`/`opacity`. Preconnect do każdego CDN.

## Pętla audytu PageSpeed (proces, nie jednorazowo)
1. Mierz: **Chrome DevTools MCP** + skill Cloudflare **web-perf** (lab), Vercel Speed Insights + PostHog Web Vitals (RUM/prod).
2. Diagnozuj: render-blocking, długie taski (Long Tasks API → Sentry), łańcuchy zależności, layout shift.
3. Popraw: `scheduler.yield()` w heavy compute, `<Suspense>` boundaries, `dynamic(ssr:false)` dla non-critical, Partytown dla third-party.
4. Alert: p75 LCP > 2.5s lub INP p75 > 300ms → zbadaj.

## A11y — WCAG 2.2 AA (minimum prawne, EAA)
- Jeden `<h1>`/route. Landmarks `<header>/<nav>/<main>/<footer>`. Skip link jako pierwszy focusable.
- Focus visible zawsze (ring z offset). Target ≥ 24×24px. Focus not obscured (sticky header nie zakrywa).
- Kontrast 4.5:1 / 3:1. Keyboard: Tab wszędzie, Escape zamyka modale, focus trap w modalach.
- `aria-live="polite"` toast/status, `assertive` błędy. `prefers-reduced-motion/-transparency`, `forced-colors` obsłużone.
- Test matrix/release: VoiceOver (Safari), NVDA (Firefox), TalkBack (Chrome). E2E a11y: axe/Playwright.

## SEO techniczne
- Metadata API w każdym `page.tsx`/`layout.tsx` (`generateMetadata`). Canonical na wszystkich stronach.
- OG image 1200×630 (`opengraph-image.tsx`/Satori). Twitter `summary_large_image`.
- JSON-LD: Organization, WebSite, Product (ecom), BreadcrumbList, Article, FAQPage, LocalBusiness (gdy lokalny).
- `app/sitemap.ts` + `app/robots.ts`. i18n: `hreflang`, `<html lang>`, `generateStaticParams` per locale.

## GEO — Generative Engine Optimization (widoczność w AI)
- **`llms.txt`** w root: mapa kluczowych treści dla modeli (ChatGPT/Perplexity/Google AI). Opcjonalnie `llms-full.txt`.
- Treść **answer-ready**: jasne nagłówki-pytania, zwięzłe definicje na początku sekcji, dane w listach/tabelach (łatwe do cytowania przez AI).
- Bogate dane strukturalne (JSON-LD FAQ/HowTo/Article) — silniki AI je preferują jako źródło.
- Encje i spójność: nazwa marki, autor, E-E-A-T (autorstwo, źródła, aktualizacje). Cytowalne fakty > marketingowy bełkot.
- Nie blokuj botów AI w `robots` jeśli chcesz widoczności; świadoma decyzja per projekt.

## Tier 2 (kopiuj)
`Syntance/moduly` → `packages/seo-geo` (`build-metadata`, `json-ld`, `llms-txt`, `robots`, `sitemap`).
