---
name: syntance-motion-3d
description: Animacje i 3D — Motion (Framer Motion), GSAP + ScrollTrigger, Lenis smooth scroll, React Three Fiber, Rive, Theatre.js. Wydajność animacji, reduced-motion, lazy-loading 3D, fallbacki na słabych urządzeniach. Włącz przy scroll animacjach, parallax, transitions, scenach 3D, cinematic.
---

# Motion i 3D

## Wybór narzędzia
- Spring, layout, presence, gesty → **Motion** (`motion/react`).
- Scroll-scrub, pin, horizontal scroll, SplitText → **GSAP** + ScrollTrigger.
- Smooth scroll → **Lenis** (`ReactLenis`).
- Scripted camera / cinematic → **Theatre.js**. Gradient na tle → ShaderGradient / OGL.
- 3D sceny/modele → **R3F** + drei + postprocessing. Interaktywna ilustracja → **Rive**.
- Lekka animacja bez scroll → Anime.js dopuszczalne (NIE do scroll-scrub).

## MUST (wydajność)
- GSAP/R3F/Lenis/Theatre NIGDY w initial bundle → `dynamic(..., { ssr: false })` PO first paint.
- Budżet JS strony bez 3D < 200 KB. Route z 3D: initial < 200 KB, three/r3f (~150 KB) lazy po first paint.
- Animuj TYLKO `transform` i `opacity`. Nie `left/top/width/height`. `will-change` tylko gdy aktywne + cleanup.
- `useEffect` z animacją → cleanup: `gsap.context()` + `ctx.revert()`. Domyślny ease `power3.out`/spring, nie `ease-in-out` bez powodu.

## Reduced motion (WCAG 2.2 + prawo)
- `prefers-reduced-motion: reduce` wyłącza parallax, smooth scroll, autoplay, ambient sound, auto-rotate 3D.
- Auto-ruch > 5s (carousel, marquee) MUSI mieć pauzę z klawiatury + widoczny przycisk.

## Detekcja słabego urządzenia (3D fallback)
- NIE polegaj na `navigator.deviceMemory` (brak w Safari/iOS).
- Kombinacja: `matchMedia('(pointer:coarse)')` + viewport < 768px + `navigator.connection?.saveData` + realny FPS probe (sliding window; < 24 fps przez 2s → PNG poster).

## Motion vs konwersja (rozstrzygnięcie)
- Animacja NIGDY nie opóźnia interaktywności primary CTA. Reveal > 600ms → CTA i tak klikalne od razu.
- Preloader to overlay NAD elementem LCP (LCP jest w DOM od razu). Brand-intro tylko 1. wizyta (`sessionStorage`).

## CSP a animacje
- Motion/GSAP wstrzykują inline `style` → `style-src` musi mieć `'unsafe-inline'` (lub hashe). `script-src` zostaje nonce + `strict-dynamic`. (szczegóły: `syntance-security`).

## A11y
- SplitText: `aria-label` z pełnym tekstem, spany `aria-hidden`.
