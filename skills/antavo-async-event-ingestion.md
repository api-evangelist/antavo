---
name: Ingest events asynchronously with an Antavo OAuth token
description: >-
  Obtain an OAuth 2.0 client-credentials access token and use the Antavo Async
  Events API to queue high-volume loyalty events, then poll for their outcome.
api: openapi/antavo-async-events-openapi.yml
operations: []
paths:
  - POST /v1/auth/token
  - POST /v1/async/events
  - GET /v1/async/events/{correlation_id}
generated: '2026-08-13'
method: generated
source: >-
  openapi/antavo-authentication-openapi.yml, openapi/antavo-async-events-openapi.yml,
  scopes/antavo-scopes.yml, https://docs.antavo.com/docs/api-settings
---

# Ingest events asynchronously with an Antavo OAuth token

The Async Events API is the **only** Antavo surface that uses OAuth 2.0 rather
than API key + Escher signature, and the only one exempt from the synchronous
rate limit. Use it for bulk ingestion, traffic spikes, and anywhere a blocked
client is worse than a delayed one.

> These operations declare no `operationId` in Antavo's published OpenAPI — 106
> of its 122 operations do not. Address them by method + path.

## Step 1 — get an authentication client

In Management UI → **API settings → Authentication Manager → Generate new Auth
Client**, set:

| Field | Notes |
|---|---|
| Name | human-readable identifier |
| Purpose | why this client exists, e.g. "sending async events requests" |
| Expiration date | after this, the client cannot mint new tokens |
| Scope | **`loyalty.async_events`** — currently the only supported scope |
| Token audience | the target service the token is valid for |

The **client secret is displayed exactly once**. Store it in a vault; it cannot
be retrieved later. The client ID stays visible in the list. Revocation is
**irreversible**.

## Step 2 — mint an access token

```
POST /v1/auth/token
Authorization: Basic base64(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&scope=loyalty.async_events
```

Response:

```json
{
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "loyalty.async_events"
}
```

`expires_in` is between **300 and 3600 seconds** and is fixed by the environment
— you cannot request a longer life. Cache the token and refresh on expiry rather
than minting one per request; every issuance is recorded in the Authentication
Manager audit log.

### Token endpoint errors

The token endpoint uses a **different envelope** from the rest of the platform:

```json
{
  "status": "error",
  "error": "invalid_scope",
  "details": {"type": "BadRequestException", "code": 220011, "message": "..."}
}
```

`error` is one of `invalid_request`, `invalid_grant`, `unauthorized_client`,
`unsupported_grant_type`, `invalid_scope`. `details.code` is drawn from
`220011`–`220015`. This is the only place in the whole Antavo surface where an
enumerated error-code set is published in the spec itself.

## Step 3 — queue an event

```
POST /v1/async/events
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "customer": "280e674c-c4ea-4a30-987a-d9267d1a5018",
  "action": "point_add",
  "data": { "points": 325 }
}
```

Returns **202** with a correlation id. The event is accepted immediately and
processed in the background.

**Only these actions are supported here** — the endpoint is a subset of
`/events`, not a drop-in replacement:

`point_add` · `checkout` · `checkout_item` · `checkout_update` ·
`checkout_update_item` · `checkout_accept` · `checkout_accept_item` ·
`checkout_reject` · `partial_refund` · `refund_item` · `refund` · `opt_in` ·
`opt_out` · `profile`

Anything else (challenges, club events, profiling, contest entries) must go
through the synchronous `POST /events`.

## Step 4 — poll for the outcome

```
GET /v1/async/events/{correlation_id}
Authorization: Bearer <access_token>
```

Once processed, the response carries the customer information and the detailed
event data.

**There is no completion callback.** The result is pull-only — Antavo will not
notify you when the event lands, so budget a polling schedule. Events are stored
reliably and processed in order even when sustained traffic exceeds capacity;
the failure mode is delay, not loss.

## Rules that will bite you

- **Still no idempotency key.** Async does not change that. A retried
  `POST /v1/async/events` produces a second event. Keep your own submission
  ledger keyed on your source system's transaction id, and check the correlation
  id you already hold before resubmitting.
- **Two credential systems, one platform.** The OAuth client here is a *separate
  object* from the workspace API key/secret used everywhere else. A bearer token
  will not authenticate `POST /events`, and an Escher signature will not
  authenticate `/v1/async/events`.
- **One scope only.** If a future Antavo release adds scopes, `loyalty.async_events`
  is what exists today — do not request a scope you have not seen documented, as
  it returns `invalid_scope`.
- **Throughput is negotiated, not published.** The docs say the async endpoint
  supports "higher request volumes" and directs capacity planning to the Antavo
  Service Desk. There is no published async rate limit.
