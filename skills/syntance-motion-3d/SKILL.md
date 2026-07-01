---
name: syntance-motion-3d
description: Animation and 3D - Motion (Framer Motion), GSAP + ScrollTrigger, Lenis smooth scroll, React Three Fiber, Rive, Theatre.js. Animation performance, reduced-motion, lazy-loading 3D, low-end device fallbacks. Activate for scroll animations, parallax, transitions, 3D scenes, cinematic sequences.
---

# Motion and 3D

## Choosing the right tool
- Spring, layout, presence, gestures → **Motion** (`motion/react`).
- Scroll-scrub, pin, horizontal scroll, SplitText → **GSAP** + ScrollTrigger.
- Smooth scroll → **Lenis** (`ReactLenis`).
- Scripted camera / cinematic → **Theatre.js**. Animated gradient background → ShaderGradient / OGL.
- 3D scenes/models → **R3F** + drei + postprocessing. Interactive illustration → **Rive**.
- Light animation without scroll → Anime.js is acceptable (NOT for scroll-scrub).

## MUST (performance)
- GSAP/R3F/Lenis/Theatre NEVER in the initial bundle → `dynamic(..., { ssr: false })` AFTER first paint.
- JS budget without 3D < 200 KB. Route with 3D: initial < 200 KB, three/r3f (~150 KB) lazy-loaded after first paint.
- Animate ONLY `transform` and `opacity`. Not `left/top/width/height`. `will-change` only while active + cleanup.
- `useEffect` with animation → cleanup with `gsap.context()` + `ctx.revert()`. Default ease `power3.out`/spring, not `ease-in-out` without reason.

## Reduced motion (WCAG 2.2 + legal)
- `prefers-reduced-motion: reduce` disables parallax, smooth scroll, autoplay, ambient sound, auto-rotating 3D.
- Auto-motion lasting > 5s (carousel, marquee) MUST have a keyboard-accessible pause + visible pause button.

## Low-end device detection (3D fallback)
- Do NOT rely on `navigator.deviceMemory` (missing in Safari/iOS).
- Combine: `matchMedia('(pointer:coarse)')` + viewport < 768px + `navigator.connection?.saveData` + a real FPS probe (sliding window; < 24 fps for 2s → fall back to a PNG poster).

## Motion vs conversion (resolution rule)
- Animation must NEVER delay the interactivity of the primary CTA. If a reveal takes > 600ms, the CTA is clickable immediately regardless.
- The preloader is an overlay ON TOP of the LCP element (the LCP element is in the DOM immediately). Brand intro only on first visit (`sessionStorage`).

## CSP and animation
- Motion/GSAP inject inline `style` → `style-src` must allow `'unsafe-inline'` (or hashes). `script-src` stays nonce + `strict-dynamic`. (details: `syntance-security`).

## A11y
- SplitText: `aria-label` with the full text, spans marked `aria-hidden`.
