---
name: syntance-frontend-stack
description: Frontend stack and architecture for Next.js 16 / React 19 - RSC vs client, rendering (SSR/SSG/ISR/PPR), library choices (Motion, GSAP, R3F, Zod, RHF, nuqs, Zustand), folder structure and naming. Activate when setting up a project, choosing a library/component, deciding on rendering strategy, or configuring next.config.
---

# Frontend Stack and Architecture

Syntance's default stack. Verify current library APIs via **context7 MCP** (never guess versions). Support component selection with the **UI/UX Pro Max** skill and **21st.dev Magic MCP** (`/ui`).

## Core
- Next.js 16 App Router + React 19 + TS strict + Tailwind v4 (OKLCH) + Shadcn (primitives).
- RSC by default. `"use client"` ONLY for: state, effects, DOM APIs, interaction, client-only libraries.
- Forms: React Hook Form + Zod + `@hookform/resolvers`. URL state: **nuqs**. Client state: **Zustand/Jotai** when Context isn't enough.

## Rendering (deliberate choice per route)
- Static content → SSG / ISR (`revalidate`) + tags (`revalidateTag`).
- Per-user / cart data → dynamic / streaming with `<Suspense>`.
- PPR (Partial Prerendering) for mixed pages: static shell + dynamic islands.
- `generateStaticParams` for `[handle]`/`[locale]`. Cache Components (`use cache`, `cacheLife`, `cacheTag`) instead of `unstable_cache`.
- Never render secrets client-side. Fetch data in RSC, not in `useEffect`.

## Libraries — what to use when
- Hover/focus/transition < 300ms → CSS/Tailwind.
- Spring, layout, presence, gestures → **Motion** (`motion/react`, formerly Framer Motion).
- Scroll-scrub, pin, horizontal scroll, SplitText → **GSAP** + ScrollTrigger.
- 3D → **React Three Fiber** + drei. Interactive illustration → **Rive**. (details: `syntance-motion-3d` skill).
- OG images → `@vercel/og` + Satori. Third-party scripts → **Partytown** (web worker).

## Where to source components (order)
1. **21st.dev Magic MCP** (`/ui "description"`) → `src/components/`.
2. Aceternity UI, Magic UI (`shadcn add <url>`), Motion Primitives, React Bits, Codrops.
Always customize visually — never ship a "bare" stock component.

## Naming / structure
- Sections: `src/components/sections/<kebab>/index.tsx` + hooks/subs alongside.
- Shadcn: `src/components/ui/`. 3D: `src/components/3d/<Scene>/index.tsx` + `Scene.glsl`.
- Components PascalCase; default export for page/layout, named for everything else. Hooks: `use` + camelCase.

## Tier 2 (copy when implementing)
Starters and patterns: `Syntance/moduly` → `apps/starter-strona`, `apps/starter-sklep`, `packages/ui`.
