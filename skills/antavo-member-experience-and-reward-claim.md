---
name: Build a headless member experience and claim a reward
description: >-
  Use the Antavo Display API to show a loyalty member what they can earn and
  spend, follow the embedded _actions affordances, and claim a reward safely.
api: openapi/antavo-display-openapi.yml
operations: []
paths:
  - GET /customers/{customer_id}/activities
  - GET /customers/{customer_id}/activities/earn
  - GET /customers/{customer_id}/activities/spend
  - GET /customers/{customer_id}/activities/rewards
  - GET /customers/{customer_id}/activities/rewards/{reward_id}
  - POST /customers/{customer_id}/activities/rewards/{reward_id}/claim
  - POST /customers/{customer_id}/activities/rewards/{reward_id}/revoke
  - GET /customers/{customer_id}/rewards
  - GET /customers/{customer_id}/coupons
  - GET /customers/{customer_id}/wallet
generated: '2026-08-13'
method: generated
source: >-
  openapi/antavo-display-openapi.yml, openapi/antavo-customer-openapi.yml,
  conventions/antavo-conventions.yml, lifecycle/antavo-lifecycle.yml
---

# Build a headless member experience and claim a reward

The Display API is Antavo's headless surface — 43 of the platform's 122
operations, all rooted at `/customers/{customer_id}`. It is what you render a
loyalty UI from, and it is the surface an agent acting for a member would use.

## Step 0 — resolve the customer

You need the **Antavo customer ID** (a UUID). If you hold your own identifier,
search first:

```
GET /customers?api_key={api_key}&{field}={value}
```

Filtering is **case-sensitive**, does **not** support partial matches, and the
field must be tagged **searchable** in the Management UI before it can be used.
`GET /customers/{customer_id}` then returns the full profile.

## Step 1 — ask what the member can do

```
GET /customers/{customer_id}/activities?api_key={api_key}
```

This aggregates every earn and spend option across all enabled modules —
challenges, gamified profiling, incentivized purchase, rewards, social follow,
social share, offline treasure hunt, contest lite, content consumption, gamified
reviews, friend referral, Instagram contests, offers, quizzes, online treasure
hunt, prize wheels and workflow campaigns.

Narrow it with `/activities/earn` or `/activities/spend` when you only need one
side of the ledger.

## Step 2 — follow `_actions`, don't hand-build URLs

Every activity object embeds an affordance map:

```json
"_actions": {
  "complete": {
    "method": "POST",
    "url": "/customers/{id}/activities/rewards/5e78c52e5553aa0e00000008/claim"
  }
}
```

This is the closest thing Antavo publishes to a state machine. Prefer it to
string-building — it tells you which transition is legal *right now* for *this*
member, which the static spec cannot.

## Step 3 — paginate correctly

`limit` and `offset` query parameters; the response carries `data` plus `next`
and `previous`.

**`next` and `previous` are relative paths**, e.g.
`/customers/{id}/activities/rewards?offset=3&limit=1` — you must join them onto
your environment base URL yourself. There is no total count, no cursor and no
RFC 5988 `Link` header.

## Step 4 — claim the reward

```
POST /customers/{customer_id}/activities/rewards/{reward_id}/claim?api_key={api_key}
Authorization: <escher signature>
```

Related operations on the same object:

- `GET .../activities/rewards/{reward_id}` — detail before claiming
- `POST .../activities/rewards/{reward_id}/bid` — bidding rewards; the amount is
  computed from the current high bid plus the configured bid step, or you may
  submit an explicit amount in the body
- `POST .../activities/rewards/{reward_id}/revoke` — reverse a claim
- `GET /customers/{customer_id}/rewards` — what they have already claimed
- `GET /customers/{customer_id}/wallet` — download URLs for assigned wallet
  passes; **a fresh access token is generated on every call**, so treat the URLs
  as short-lived and never cache them

## Rules that will bite you

- **A claim spends real points and is not idempotent.** There is no idempotency
  key on this or any other Antavo write. If the call times out, read
  `GET /customers/{customer_id}/rewards` to see whether the claim landed
  **before** you retry. An agent should require explicit human confirmation
  before claiming, and must never auto-retry.
- **Use the Display API, not the legacy Rewards API.**
  `POST /rewards/{reward_id}/claim` is explicitly deprecated — "This endpoint is
  not supported anymore" — and the whole Rewards API is marked *LEGACY* in
  Antavo's own API index. Note that **no OpenAPI operation carries
  `deprecated: true`**, so this is invisible to tooling; it is only in prose.
- **Challenges have a v2.** Prefer
  `GET /v2/customers/{customer_id}/activities/challenges` and
  `GET /v2/customers/{customer_id}/challenges` — the v1 pair's own descriptions
  point you at v2 for filtering and a more complete view.
- **`-` is a wildcard, not a placeholder.** `GET /customers/-/events` returns
  events for *all* customers. Sending a literal `-` where you meant a customer id
  will silently widen the query across the whole workspace.
- **Everything here is member PII.** Profiles, purchase history and custom
  attributes come back on most reads. Antavo is ISO 27018 and GDPR compliant;
  carry that posture through to wherever you render or store the response.
- **Module-not-enabled looks like not-found.** A `404` may mean the reward does
  not exist, or that the module owning the endpoint is not enabled for this
  workspace. Custom login, points economy and most gamification surfaces are
  optional modules.

## Errors

`{"error": {"type": "...", "code": <int>, "message": "..."}}` — plain JSON, not
RFC 9457. See `errors/antavo-problem-types.yml`. `403` is typically the workspace
IP filter or signature enforcement, not an authorisation decision about the
member.
