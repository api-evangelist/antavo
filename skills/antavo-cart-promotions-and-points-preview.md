---
name: Preview points and apply cart promotions at checkout
description: >-
  Simulate the loyalty points and promotional discounts a cart would earn before
  committing, then finalize the checkout through the Antavo Promotion Engine.
api: openapi/antavo-promotion-engine-openapi.yml
operations: []
paths:
  - POST /extensions/automation/campaign-bonus
  - POST /offers/
  - POST /v1/cart
  - POST /v1/cart/finalize
  - GET /v1/promotions
  - GET /v1/promotion/{promotionId}
  - POST /v1/promotion/{promotionId}/status
generated: '2026-08-13'
method: generated
source: >-
  openapi/antavo-promotion-engine-openapi.yml, openapi/antavo-points-preview-openapi.yml,
  openapi/antavo-offers-openapi.yml, conventions/antavo-conventions.yml,
  https://developers.antavo.com/docs/implementing-promotion-engine
---

# Preview points and apply cart promotions at checkout

This is the pre-purchase half of Antavo: everything you can compute about a cart
*before* the customer commits. Three separate surfaces cover it, on **two
different hosts**.

> **Host warning.** The Promotion Engine is served from a different base than the
> rest of the platform. Antavo publishes `https://promotion.test.antavo.com/api`
> in that spec's `servers[]`; the Loyalty Engine surfaces (`/offers/`,
> `/extensions/automation/campaign-bonus`) sit on the main API host
> (`https://api.antavo.com`, staging `https://api.staging.antavo.com`). Confirm
> your production Promotion Engine base with Antavo — do not assume it matches
> the main API host.

## Step 1 — preview the points

```
POST /extensions/automation/campaign-bonus?api_key={api_key}
Authorization: <escher signature>
```

Returns the points a transaction would earn, **including the bonus points
assigned by Campaign bonus workflow nodes**. Works with standard and
multi-account configurations.

This is the safest operation on the entire Antavo surface: a **read-only
simulation of a write**. If an agent needs to tell a customer "this purchase
earns you 340 points", this is the call — not a speculative `checkout` event.

Formerly known as the **Workflow Campaigns API**; the Management UI still lists
the rename.

## Step 2 — check for pre-purchase offers

```
POST /offers/?api_key={api_key}

{
  "cart": { ... },            // same shape as an Antavo checkout event
  "customer": "<antavo customer id>",
  "store": "<antavo store id>",
  "eligible_only": true
}
```

`store` may yield store-specific offers when configured. `eligible_only`
restricts the response to offers this customer can actually take. The member-side
view of the same data is `GET /customers/{customer_id}/activities/offers`, and a
claim is `POST /customers/{customer_id}/activities/offers/{offer_id}/claim`.

## Step 3 — evaluate promotions on the cart

```
POST /v1/cart
Authorization: Bearer <token>
```

Returns the cart object, all matched promotions, the exact discounts applied,
adjusted per-unit prices after discount, updated tax amounts and total discount
values.

**This call has a side effect.** If a matched promotion carries an application
limit and a locking period, submitting the cart takes a **reservation**. The
reservation is released automatically only when the lock expires. So:

- The same `cartId` may be resubmitted to *recalculate* as the cart changes —
  that is the intended pattern.
- Do not fan out speculative `/v1/cart` calls across variant carts for the same
  customer; each one can lock promotion capacity that other customers then cannot
  use.

## Step 4 — finalize

```
POST /v1/cart/finalize
```

Same calculation as `/v1/cart`, plus:

- promotions are **applied and counted** against their application limits
- the cart **cannot be resubmitted with the same `cartId`**

You may call `/v1/cart/finalize` directly without `/v1/cart` first when you do
not need the simulation.

## Managing the promotion catalogue

- `GET /v1/promotions` — every promotion configured in the workspace, with full
  settings. No parameters required.
- `GET /v1/promotion/{promotionId}` — one promotion in detail.
- `POST /v1/promotion/{promotionId}/status` — activate, return to draft, or
  archive. **Archiving is terminal**: "once a promotion is archived, it cannot be
  changed back to any other state." Never let an agent archive without explicit
  human confirmation.

## Rules that will bite you

- **`/v1/cart` is not a pure function.** Treat it as a write with a reservation
  side effect, not as a quote endpoint. `/extensions/automation/campaign-bonus`
  is the genuinely side-effect-free preview.
- **`cartId` is a one-shot key at finalize.** Reusing it after finalize is
  rejected — but this is *not* an idempotency key. It will not make a retried
  finalize safe in the general case, and Antavo publishes no idempotency
  mechanism anywhere.
- **Two auth models on one flow.** The Promotion Engine spec declares
  `bearerAuth` (JWT); the Loyalty Engine surfaces in steps 1 and 2 use
  `api_key` + Escher signature. A single checkout integration needs both
  credentials.
- **Rate limits are shared and silent.** All of this counts against the same per
  API key budget (1,500 req/min shared stack, 20,000 dedicated) with **no
  `RateLimit-*` or `Retry-After` header** to read. Do not poll.
- **Cart shape follows the checkout event.** The `cart` object in `/offers/`
  mirrors the Antavo `checkout` event format — see
  <https://developers.antavo.com/docs/api-events>. Reuse one serializer for both
  so a promotion preview and the eventual `checkout` event cannot disagree.
