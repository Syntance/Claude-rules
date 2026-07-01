---
name: syntance-design
description: Awwwards-caliber visual design - OKLCH color systems, typography, layout, hierarchy, hero moments, icons/assets, copywriting and microcopy. Zero-stock principle. Activate when designing visuals, choosing a palette, typography, hero sections, sourcing assets, or writing UI copy.
---

# Design System and Aesthetics

Aesthetic benchmark: Active Theory, Resn, Locomotive, Obys, Igloo Inc., Basement, Immersive Garden. Use the **UI/UX Pro Max** skill (`/ui-ux-pro-max`) for palettes and type pairings, and **21st.dev Magic** (`/ui`) for ready-made sections.

## Mindset
- Craft > feature count. Restraint > clutter. One memorable moment per project; everything else supports it.
- Specificity tax: the more generic, the worse. Every element must answer "why this, specifically here".
- First frame contract: what the user sees after the loader decides everything. Value prop readable in 5s.

## Color (OKLCH)
- Palette as CSS variables, OKLCH color space. Hex only when an external API requires it, with `@supports (color: oklch(0 0 0))` fallback.
- NEVER pure `#000`/`#FFF` in brand elements — always a slightly toned OKLCH (exception: data tables, print, contrast requirements).
- Contrast: 4.5:1 normal text, 3:1 large text + UI. Check every pair.

## Typography
- No stock fonts without justified variable axes (Roboto, Open Sans, Poppins, Lato, Montserrat, Inter "out of the box").
- `next/font`, display: swap, preload only for display font. Modular scale, deliberate tracking/leading.

## Layout and hierarchy
- Grid + vertical rhythm. One dominant element per view. Whitespace as a tool, not absence of content.
- Hero shows the product/outcome (the "after state"), not an abstraction. One primary CTA, visually dominant.

## Zero-stock (NEVER)
- Stock icons everywhere (Lucide/Heroicons) in hero/nav without customization → use Iconsax or a custom set of ≥ 12 icons.
- Stock illustrations (unDraw, Storyset, Humaaans). Stock photos without grading (LUT, contrast curve).
- Generic Tailwind gradients (`from-purple-500 to-pink-500`).

## Copy / microcopy
- No generic phrases ("Get Started", "Learn More", "Our Services"). Verb + outcome.
- Microcopy next to CTAs disarms risk: "No card required", "Cancel anytime", "Reply within 24h".
- Alt text: decorative images `alt=""`, meaningful ones describe intent.

## Assets
- Images: AVIF/WebP via next/image. LCP image ≤ 120 KB. User-uploaded SVG forbidden (XSS risk).
- Video: poster + lazy load, `prefers-reduced-motion` disables autoplay.

Related: `syntance-motion-3d` (motion), `syntance-conversion-cro` (layout for conversion), `syntance-perf-a11y-seo-geo` (contrast/a11y).
