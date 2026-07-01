---
name: syntance-frontend-stack
description: Dobór stacku i architektury frontendu Next.js 16 / React 19 — RSC vs client, rendering (SSR/SSG/ISR/PPR), dobór bibliotek (Motion, GSAP, R3F, Zod, RHF, nuqs, Zustand), struktura folderów i naming. Włącz przy zakładaniu projektu, wyborze biblioteki/komponentu, decyzji o renderowaniu, konfiguracji next.config.
---

# Frontend stack i architektura

Domyślny stack Syntance. Aktualne API bibliotek weryfikuj przez **context7 MCP** (nie zgaduj wersji). Dobór komponentów wspieraj skillem **UI/UX Pro Max** i **21st.dev Magic MCP** (`/ui`).

## Core
- Next.js 16 App Router + React 19 + TS strict + Tailwind v4 (OKLCH) + Shadcn (prymitywy).
- RSC domyślnie. `"use client"` TYLKO dla: stanu, efektu, DOM API, interakcji, bibliotek client-only.
- Formularze: React Hook Form + Zod + `@hookform/resolvers`. URL state: **nuqs**. Client state: **Zustand/Jotai** gdy Context nie wystarcza.

## Rendering (wybór świadomy per route)
- Statyczny content → SSG / ISR (`revalidate`) + tagi (`revalidateTag`).
- Dane per-user / koszyk → dynamic / streaming z `<Suspense>`.
- PPR (Partial Prerendering) dla stron mieszanych: statyczna skorupa + dynamiczne wyspy.
- `generateStaticParams` dla `[handle]`/`[locale]`. Cache Components (`use cache`, `cacheLife`, `cacheTag`) zamiast `unstable_cache`.
- Nigdy nie renderuj sekretów po stronie klienta. Fetch danych w RSC, nie w `useEffect`.

## Biblioteki — kiedy czego
- Hover/focus/transition < 300ms → CSS/Tailwind.
- Spring, layout, presence, gesty → **Motion** (`motion/react`, dawniej Framer Motion).
- Scroll-scrub, pin, horizontal, SplitText → **GSAP** + ScrollTrigger.
- 3D → **React Three Fiber** + drei. Interaktywna ilustracja → **Rive**. (szczegóły: skill `syntance-motion-3d`).
- OG images → `@vercel/og` + Satori. Third-party scripts → **Partytown** (web worker).

## Skąd brać komponenty (kolejność)
1. **21st.dev Magic MCP** (`/ui "opis"`) → `src/components/`.
2. Aceternity UI, Magic UI (`shadcn add <url>`), Motion Primitives, React Bits, Codrops.
Zawsze dostosuj wizualnie — zero „gołego” komponentu ze stocka.

## Naming / struktura
- Sekcje: `src/components/sections/<kebab>/index.tsx` + hooks/subs obok.
- Shadcn: `src/components/ui/`. 3D: `src/components/3d/<Scene>/index.tsx` + `Scene.glsl`.
- Komponenty PascalCase; default export dla page/layout, named dla reszty. Hooks: `use` + camelCase.

## Tier 2 (kopiuj gdy implementujesz)
Startery i wzorce: `Syntance/moduly` → `apps/starter-strona`, `apps/starter-sklep`, `packages/ui`.
