---
name: syntance-design
description: Design wizualny klasy Awwwards — system kolorów OKLCH, typografia, layout, hierarchia, hero moment, ikony/assety, copywriting i mikrocopy. Zasada zero-stock. Włącz przy projektowaniu wyglądu, palety, typografii, sekcji hero, doborze assetów, pisaniu treści UI.
---

# Design system i estetyka

Benchmark estetyki: Active Theory, Resn, Locomotive, Obys, Igloo Inc., Basement, Immersive Garden. Do palet, par typograficznych i wzorców używaj skilla **UI/UX Pro Max** (`/ui-ux-pro-max`). Do gotowych sekcji **21st.dev Magic** (`/ui`).

## Mindset
- Craft > liczba featur'ów. Restraint > przeładowanie. Jeden pamiętny moment na projekt, reszta go wspiera.
- Specificity tax: im bardziej generycznie, tym gorzej. Każdy element odpowiada „czemu akurat to, akurat tu”.
- First frame contract: co user widzi po loaderze decyduje o wszystkim. Value prop czytelna w 5s.

## Kolor (OKLCH)
- Paleta w CSS variables, przestrzeń OKLCH. Hex tylko gdy zewnętrzne API wymaga, z `@supports (color: oklch(0 0 0))` fallback.
- NIGDY czystego `#000`/`#FFF` w brandzie — zawsze lekko tonowany OKLCH (wyjątek: tabele danych, print, wymóg kontrastu).
- Kontrast: 4.5:1 tekst normal, 3:1 large + UI. Sprawdzaj każdą parę.

## Typografia
- Bez stock fontów bez uzasadnienia axes (Roboto, Open Sans, Poppins, Lato, Montserrat, Inter „z pudełka”).
- `next/font`, display: swap, preload tylko dla display. Skala modularna, świadomy tracking/leading.

## Layout i hierarchia
- Siatka + rytm pionowy. Jedna dominanta na widok. Whitespace jako narzędzie, nie brak treści.
- Hero pokazuje produkt/wynik („after state”), nie abstrakcję. Jedno primary CTA dominujące wizualnie.

## Zero-stock (NEVER)
- Stock ikony wszędzie (Lucide/Heroicons) w hero/nav bez override → Iconsax lub custom set ≥ 12 ikon.
- Stock ilustracje (unDraw, Storyset, Humaaans). Stock zdjęcia bez gradingu (LUT, contrast curve).
- Generyczne gradienty Tailwind (`from-purple-500 to-pink-500`).

## Copy / mikrocopy
- Zero generyków („Get Started”, „Learn More”, „Our Services”). Czasownik + wynik.
- Mikrocopy rozbraja ryzyko przy CTA: „Bez karty”, „Anuluj w każdej chwili”, „Odpowiedź w 24h”.
- Alt texty: dekoracyjne `alt=""`, znaczące opisują intencję.

## Assety
- Obrazy: AVIF/WebP przez next/image. LCP image ≤ 120 KB. SVG od usera zakazany (XSS).
- Video: poster + lazy, `prefers-reduced-motion` wyłącza autoplay.

Powiązane: `syntance-motion-3d` (ruch), `syntance-conversion-cro` (układ pod konwersję), `syntance-perf-a11y-seo-geo` (kontrast/a11y).
