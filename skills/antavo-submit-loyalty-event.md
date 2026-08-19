---
name: Submit a loyalty event to Antavo
description: >-
  Record a customer interaction (purchase, point award, opt-in) on an Antavo
  loyalty member's event history, using signed requests, and verify it landed.
api: openapi/antavo-events-openapi.yml
operations:
  - events            # POST /events
  - bulk              # POST /events/bulk
paths:
  - POST /events
  - POST /events/bulk
  - GET /customers/{customer_id}/events
generated: '2026-08-13'
method: generated
source: >-
  openapi/antavo-events-openapi.yml, openapi/antavo-display-openapi.yml,
  conventions/antavo-conventions.yml, errors/antavo-problem-types.yml
---

# Submit a loyalty event to Antavo

Events are the centre of the Antavo Loyalty Cloud. Everything else — points,
tiers, challenges, transactions, workflow campaigns — is computed from the event
stream. Writing an event is therefore a **consequential, non-reversible** action.

## Before you call

1. **You need a provisioned workspace.** There is no self-service signup, no
   public sandbox and no demo key. The `api_key` and `secret` are generated in
   Management UI → API settings for one workspace in one environment. See
   `sandbox/antavo-sandbox.yml`.
2. **Pick the right base URL.**
   - Production: `https://api.antavo.com`
   - Staging: `https://api.staging.antavo.com` (this is what the published
     `servers[]` block names)
3. **Sign the request.** Key-based endpoints use Escher (an AWS SigV4-derived
   scheme) in the `Authorization` header. Production requires it today; **from
   2026-12-31 every environment requires it** (`changelog/antavo-changelog.yml`).
   Do not compute signatures in front-end code. Use `antavo/escher-php`,
   `@antavo/api-signature-node`, or the pre-request script in Antavo's public
   Postman collection.
4. **The `api_key` travels as a QUERY PARAMETER**, not a header. Assume it is
   visible in every proxy and access log; the signature, not the key, is what
   authenticates the request.

## Step 1 — know which action you are recording

The `action` field is a fixed vocabulary published at
<https://developers.antavo.com/docs/api-events>. Common ones:

`point_add` · `point_spend` · `point_unspend` · `checkout` · `checkout_item` ·
`checkout_update` · `checkout_accept` · `checkout_reject` · `refund` ·
`partial_refund` · `opt_in` · `opt_out` · `profile`

Do not invent an action name. An unrecognised action is rejected, and a
*misspelled* one is worse — Antavo's own published example contains the typo
`"action": "cehckout"`, which is exactly the failure mode to guard against.

## Step 2 — submit a single event

```
POST /events?api_key={api_key}
Authorization: <escher signature>
Content-Type: application/json

{
  "customer": "280e674c-c4ea-4a30-987a-d9267d1a5018",
  "action": "point_add",
  "data": { "points": 325 }
}
```

Variants Antavo documents:

- **External ID** — add `"external_id": "..."` to link the event to the
  customer's identifier in your own system.
- **Multi-account** — add `"account": "main_account"`. Requires the Points
  economy module. With no `account`, the event lands on the default account.
- **Guest checkout** — send `"guest": "true"` on a single `/events` call, then
  link it to a member later with a `checkout_claim` event. Guest checkout is
  **not supported** on the bulk endpoint.

## Step 3 — or submit a batch

```
POST /events/bulk?api_key={api_key}
```

Events are processed **individually and synchronously**. The response returns a
success message or an error **per event, in submission order**. A top-level error
code is only returned when the *whole request* is invalid — a per-event failure
does not fail the batch, so you must read every element of the response array.

## Step 4 — verify it landed

```
GET /customers/{customer_id}/events?api_key={api_key}&limit=10
```

Filter by action and paginate with `limit`/`offset`. This is the same endpoint
historically named `/customers/{customer_id}/history`; both paths still work.

## Rules that will bite you

- **There is no idempotency key.** Antavo publishes none. If you retry a
  `POST /events` after a timeout you will very likely create a **second** event
  and award the points twice. Before any retry, read the customer's event stream
  and confirm whether the first attempt landed. Never retry blindly, and never
  wrap this call in an automatic retry policy.
- **Order matters and is never corrected.** Events are processed FIFO in the
  order received. An event resubmitted after an error is stamped with the time it
  finally succeeded, not when it happened.
- **Throttle per customer.** Antavo explicitly asks callers to space out
  consecutive requests affecting the same customer.
- **Rate limits are invisible at runtime.** 1,500 req/min on a shared stack,
  20,000 req/min dedicated, per API key, cumulative across all endpoints except
  Async Events. **No `RateLimit-*` or `Retry-After` header is returned** — you
  cannot read your remaining budget, only observe a 429. Back off exponentially.
- **Tolerate unknown response fields.** Workspace-configured custom attributes
  and translation objects appear in responses without being in any spec.

## Errors

Antavo does not use RFC 9457. Errors are:

```json
{"error": {"type": "NotFoundException", "code": 0, "message": "Not Found"}}
```

The numeric `code` is the stable machine key; the `message` string is not.
A `404` may mean *the record does not exist* **or** *the module is not enabled for
this workspace* — do not treat it as proof of absence. A `403` usually means the
workspace IP filter rejected your source address, or signature enforcement is on
for that endpoint and you sent a bare key. Full catalogue in
`errors/antavo-problem-types.yml`.

## When to use the async endpoint instead

For high-volume ingestion use `POST /v1/async/events`, which is exempt from the
synchronous rate limit, acknowledges immediately, and returns a correlation id you
poll at `GET /v1/async/events/{correlation_id}`. It uses **OAuth 2.0 client
credentials** with scope `loyalty.async_events`, not key+signature — a different
credential entirely. See `skills/antavo-async-event-ingestion.md`.
