---
name: syntance-perf-a11y-seo-geo
description: Performance (Core Web Vitals, PageSpeed Insights), accessibility (WCAG 2.2 AA), technical SEO, and GEO (Generative Engine Optimization - llms.txt, JSON-LD, AI-answer-ready content). Activate when optimizing speed, auditing Lighthouse/PageSpeed, working on a11y, metadata, sitemap/robots, or visibility in search engines and AI engines.
---

# Performance, A11y, SEO, GEO

## Core Web Vitals (hard limits)
- LCP < 2.0s (target 1.5s, includes the preloader). CLS < 0.05. INP < 200ms (target 100ms). FCP < 1.5s. TTFB < 500ms.
- Initial JS < 200 KB gz/route (3D lazy-loaded after first paint). Check with `@next/bundle-analyzer`.
- LCP image ≤ 120 KB AVIF, next/image with `priority` + `fetchPriority="high"` + `sizes`. Fonts: next/font `display:swap`.
- Animate only `transform`/`opacity`. Preconnect to every CDN.

## PageSpeed audit loop (a process, not a one-off)
1. Measure: **Chrome DevTools MCP** + Cloudflare **web-perf** skill (lab), Vercel Speed Insights + PostHog Web Vitals (RUM/production).
2. Diagnose: render-blocking resources, long tasks (Long Tasks API → Sentry), dependency chains, layout shift.
3. Fix: `scheduler.yield()` in heavy compute, `<Suspense>` boundaries, `dynamic(ssr:false)` for non-critical code, Partytown for third-party scripts.
4. Alert: p75 LCP > 2.5s or INP p75 > 300ms → investigate.

## A11y — WCAG 2.2 AA (legal minimum, EU Accessibility Act)
- One `<h1>`/route. Landmarks `<header>/<nav>/<main>/<footer>`. Skip link as the first focusable element.
- Focus always visible (ring with offset). Target ≥ 24×24px. Focus not obscured (sticky header must not cover the focused element).
- Contrast 4.5:1 / 3:1. Keyboard: Tab reaches everything, Escape closes modals, focus trap inside modals.
- `aria-live="polite"` for toasts/status, `assertive` for errors. Respect `prefers-reduced-motion/-transparency` and `forced-colors`.
- Screen reader test matrix per release: VoiceOver (Safari), NVDA (Firefox), TalkBack (Chrome). Automated a11y E2E: axe/Playwright.

## Technical SEO
- Metadata API in every `page.tsx`/`layout.tsx` (`generateMetadata`). Canonical tag on every page.
- OG image 1200×630 (`opengraph-image.tsx`/Satori). Twitter `summary_large_image`.
- JSON-LD: Organization, WebSite, Product (ecommerce), BreadcrumbList, Article, FAQPage, LocalBusiness (when applicable).
- `app/sitemap.ts` + `app/robots.ts`. i18n: `hreflang`, `<html lang>`, `generateStaticParams` per locale.

## GEO — Generative Engine Optimization (AI visibility)
- **`llms.txt`** at the root: a map of key content for AI models (ChatGPT/Perplexity/Google AI Overviews). Optionally `llms-full.txt`.
- **Answer-ready** content: clear question-headings, concise definitions/answers at the start of each section, data in lists/tables (easy for AI to cite).
- Rich structured data (JSON-LD FAQ/HowTo/Article) — AI engines prefer these as sources.
- Entities and consistency: brand name, authorship, E-E-A-T (authorship, sources, update dates). Citable facts > marketing fluff.
- Deliberately decide whether to allow AI bots in `robots` — a conscious per-project choice, not a default block.

## Tier 2 (copy)
`Syntance/moduly` → `packages/seo-geo` (`build-metadata`, `json-ld`, `llms-txt`, `robots`, `sitemap`).
