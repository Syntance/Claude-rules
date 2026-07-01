---
name: syntance-conversion-cro
description: Konwersja i CRO — struktura strony pod cel, hero, dyscyplina CTA, redukcja tarcia w formularzach, dowód i zaufanie, event taxonomy, A/B testy, funnel. Włącz przy projektowaniu landingu, PDP, pricingu, formularzy, ścieżki zakupowej, pomiaru konwersji.
---

# Konwersja i CRO

## Strategia przed pikselem
- Zdefiniuj PRZED kodem: ICP (kto), JTBD (po co), value proposition (1 zdanie), primary conversion action per strona, north-star metric.
- Mapa funnela: awareness → interest → decision → action. Każda sekcja popycha do następnego kroku.
- Jeden cel = jedna primary akcja per widok. (strategia B2B: skill `syntance-strategia-negacz`).

## Hero
- Value prop w 5s: CO, DLA KOGO, DLACZEGO lepsze. Nie hasło-zagadka.
- Jedno primary CTA above the fold (kontrast + rozmiar + pozycja). Secondary wyraźnie słabsze (ghost/link).

## CTA
- Czasownik + wynik: „Zobacz jak zbudowaliśmy X”, „Załóż konto w 2 min”.
- Powtarzaj TO SAMO CTA w dół strony po sekcjach-dowodach, nie mnóż różnych.
- Fitts's law: duże, w dolnej 1/3 na mobile, target ≥ 44×44px. Mikrocopy rozbraja ryzyko.

## Redukcja tarcia (formularze)
- Każde pole = koszt. Tylko niezbędne, email-first, reszta progresywnie.
- Jedna kolumna, label nad polem, walidacja inline po `blur` (nie po submit), `autocomplete`, `inputmode`/`type`.
- Guest checkout zawsze dostępny; konto opcjonalne PO zakupie.
- Błędy dostępne: `aria-live`, focus na pierwszym błędzie (inaczej user porzuca).

## Dowód i zaufanie
- Social proof blisko CTA: liczby konkretne, logo, testimoniale z twarzą + nazwiskiem + rolą.
- Trust signals w checkout/pricing: zabezpieczenia płatności, polityka zwrotów, realny kontakt.
- Urgency/scarcity TYLKO prawdziwa. Fake urgency = dark pattern, zakazane (DSA → `syntance-legal-pl-eu`).

## Pomiar i iteracja
- Event taxonomy z góry: `cta_click`, `form_start`, `form_submit`, `checkout_step`, `purchase`. Spójne, dokumentowane.
- Funnel + conversion rate + session replay (sampled) na drop-off. A/B przez feature flags na hipotezach.
- Mierz na realnym ruchu (RUM), nie na opiniach. Perf = funkcja konwersji (każde 100ms LCP to spadek).

## Trade-off estetyka ↔ konwersja
- Na stronach transakcyjnych (pricing/checkout/PDP) wygrywa jasność. „Wow” dostaje przestrzeń na hero/case studies/about.
