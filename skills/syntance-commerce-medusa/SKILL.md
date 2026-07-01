---
name: syntance-commerce-medusa
description: E-commerce na Medusa v2 — architektura sklepu, Medusa JS SDK, storefront Next.js, koszyk i stan, PLP/PDP, katalog, inventory, wysyłka (InPost/DPD), pipeline zamówień, Meilisearch, design sklepu. Włącz przy budowie sklepu, integracji Medusa, koszyku, kartach produktu, wysyłce, wyszukiwarce produktów.
---

# E-commerce (Medusa v2)

Benchmark: Aimé Leon Dore, Kith, APC, Frankie Shop, Allbirds, Gymshark. Checkout ma osobny twardy standard → `syntance-checkout-payments`.

## Architektura
- Backend Medusa v2 (API) + storefront Next.js 16. Nie mieszaj logiki commerce do storefrontu — konsumuj przez SDK.
- **Medusa JS SDK** (`@medusajs/js-sdk`) do storefrontu. `QueryService` + filters po stronie backendu (nigdy raw SQL z user inputem).
- Moduły domenowe w `apps/backend/src/modules/<domena>` (własne serwisy, modele, migracje).

## Koszyk i stan
- Koszyk server-authoritative (Medusa cart). Client trzyma tylko cart id + optimistic UI. Rewalidacja po mutacji.
- Region/waluta z kontekstu (nie hardkoduj). Ceny liczy backend, front tylko wyświetla.

## PLP / PDP
- PLP: SSG/ISR + filtry przez **nuqs** (URL state). Paginacja/infinite z zachowaniem scrolla.
- PDP: JSON-LD Product (cena, dostępność, rating), warianty, `min_order_quantity`, galeria z LCP-priority.
- Wyszukiwarka: **Meilisearch** (moduł + client). Indeks synchronizowany subscriberem.

## Inventory / wysyłka
- Stan magazynowy jako źródło prawdy; „scarcity” tylko realny. Rezerwacja przy checkoutcie.
- Wysyłka PL: InPost (paczkomaty — mapa/geowidget), DPD (moduł fulfillment). Koszty widoczne PRZED ostatnim krokiem checkoutu.

## Pipeline zamówień
- `order.placed` → mail idempotentny, powiadomienie sklepu, sync statusu. Odstąpienie 14 dni + zwroty (→ `syntance-legal-pl-eu`, moduł `returns`).
- Rewalidacja storefrontu po zmianie produktu/treści (subscriber → `revalidateTag`).

## Tier 2 (kopiuj — nie pisz od zera)
`Syntance/moduly` → `packages/commerce` (cart, checkout, medusa client, search), `apps/backend/src/modules` (przelewy24, tpay, dpd-fulfillment, meilisearch, returns), `apps/starter-sklep`.
Instalacja gotowego sklepu → `syntance-moduly-deploy`.
