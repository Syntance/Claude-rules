---
name: syntance-conversion-cro
description: Conversion and CRO - page structure by goal, hero design, CTA discipline, form friction reduction, trust and proof, event taxonomy, A/B testing, funnels. Activate when designing a landing page, PDP, pricing page, forms, or the purchase path, or measuring conversion.
---

# Conversion and CRO

## Strategy before pixels
- Define BEFORE writing code: ICP (who), JTBD (why they come), value proposition (one sentence), primary conversion action per page, north-star metric.
- Funnel map: awareness → interest → decision → action. Every section pushes toward the next step.
- One goal = one primary action per view. (B2B strategy: `syntance-strategia-negacz` skill).

## Hero
- Value prop readable in 5s: WHAT, FOR WHOM, WHY better. Not a riddle-slogan.
- One primary CTA above the fold (contrast + size + position). Secondary CTA clearly weaker (ghost/link).

## CTA
- Verb + outcome: "See how we built X", "Create an account in 2 min".
- Repeat the SAME CTA down the page after proof sections, don't multiply different ones.
- Fitts's law: large, in the bottom third on mobile, target ≥ 44×44px. Microcopy disarms risk.

## Reducing friction (forms)
- Every field is a cost. Only essentials, email-first, everything else progressive.
- Single column, label above field, inline validation on `blur` (not on submit), `autocomplete`, correct `inputmode`/`type`.
- Guest checkout always available; account optional AFTER purchase.
- Accessible errors: `aria-live`, focus on the first error (otherwise the user abandons).

## Proof and trust
- Social proof near the CTA: concrete numbers, logos, testimonials with a face + name + role.
- Trust signals at checkout/pricing: payment security, return policy, real contact info.
- Urgency/scarcity ONLY when genuine. Fake urgency = dark pattern, banned (DSA → `syntance-legal-pl-eu`).

## Measurement and iteration
- Define event taxonomy upfront: `cta_click`, `form_start`, `form_submit`, `checkout_step`, `purchase`. Consistent, documented.
- Funnel + conversion rate + session replay (sampled) on drop-off points. A/B test via feature flags on hypotheses, not randomly.
- Measure on real traffic (RUM), not opinions. Perf = a function of conversion (every 100ms of LCP is a measurable drop).

## Aesthetics vs conversion trade-off
- On transactional pages (pricing/checkout/PDP), clarity wins. "Wow" gets room on hero/case studies/about pages.
