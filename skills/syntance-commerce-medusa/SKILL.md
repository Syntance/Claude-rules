---
name: syntance-commerce-medusa
description: E-commerce on Medusa v2 - store architecture, Medusa JS SDK, Next.js storefront, cart and state, PLP/PDP, catalog, inventory, shipping (courier integrations), order pipeline, Meilisearch, store design. Activate when building a store, integrating Medusa, working on the cart, product pages, shipping, or product search.
---

# E-Commerce (Medusa v2)

Benchmark: Aimé Leon Dore, Kith, APC, Frankie Shop, Allbirds, Gymshark. Checkout has its own hard standard → `syntance-checkout-payments`.

## Architecture
- Medusa v2 backend (API) + Next.js 16 storefront. Don't mix commerce logic into the storefront — consume it via the SDK.
- **Medusa JS SDK** (`@medusajs/js-sdk`) in the storefront. `QueryService` + filters on the backend side (never raw SQL with user input).
- Domain modules under `apps/backend/src/modules/<domain>` (own services, models, migrations).

## Cart and state
- Cart is server-authoritative (Medusa cart). Client only holds the cart id + optimistic UI. Revalidate after every mutation.
- Region/currency come from context (never hardcoded). Backend computes prices; the frontend only displays them.

## PLP / PDP
- PLP: SSG/ISR + filters via **nuqs** (URL state). Pagination/infinite scroll that preserves scroll position.
- PDP: JSON-LD Product (price, availability, rating), variants, `min_order_quantity`, LCP-priority gallery image.
- Search: **Meilisearch** (module + client). Index kept in sync via a subscriber.

## Inventory / shipping
- Inventory state as the single source of truth; "scarcity" claims must be real. Reserve stock at checkout.
- Domestic shipping: parcel-locker networks and courier modules (fulfillment providers registered per project). Shipping cost shown BEFORE the final checkout step.

## Order pipeline
- `order.placed` → idempotent email, store notification, status sync. 14-day withdrawal + returns (→ `syntance-legal-pl-eu`, `returns` module).
- Revalidate the storefront after product/content changes (subscriber → `revalidateTag`).

## Tier 2 (copy — don't build from scratch)
`Syntance/moduly` → `packages/commerce` (cart, checkout, Medusa client, search), `apps/backend/src/modules` (payment providers, fulfillment, Meilisearch, returns), `apps/starter-sklep`.
Deploying a ready-made store → `syntance-moduly-deploy`.
