# Antavo (antavo)

Antavo is an enterprise loyalty management platform - the **Antavo AI Loyalty Cloud** - that lets brands build and run omnichannel, multi-brand, multi-country loyalty programs. Its API-first, headless Loyalty Engine exposes a comprehensive REST API over HTTPS covering customer events, customer profiles, the headless Display surface for member-facing loyalty experiences, configurable entities (rewards, challenges, stores, products, transactions), coupons, offers, leaderboards, clubs, promotions, and bulk operations. Requests use standard HTTP verbs (GET, POST, PUT, DELETE) with URL-encoded or JSON bodies and JSON responses, secured by an API key and secret with optional request signing (HMAC), IP filtering, and token-based authentication.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/antavo/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/antavo/refs/heads/main/apis.yml)

## Access Model

Antavo's developer documentation (developers.antavo.com, apidocs.antavo.com, docs.antavo.com) is fully public and openly browsable. Live API access, however, is **provisioned per Antavo environment for enterprise customers** - an API key and secret are generated in the platform's API settings (the secret is shown only once), and the API host is environment-specific, of the form `https://api.<environment>.antavo.com`. Antavo is an enterprise-only product with custom, contact-sales pricing; there is no self-service public sandbox. The endpoints documented in this entry are therefore drawn from Antavo's public reference and honestly modeled rather than exercised live against a provisioned account.

## Tags

- Loyalty
- Customer Loyalty
- Rewards
- Enterprise
- Headless
- Retail
- Marketing
- Engagement

## Timestamps

- **Created:** 2026-07-10
- **Modified:** 2026-07-10

## APIs

### Antavo Events API
Records customer interactions from e-commerce, POS, websites, and mobile apps as loyalty events (for example `point_add`, `checkout_accept`), driving the rules and workflows of the loyalty program. Supports single and bulk event submission and reading a customer's event history.

### Antavo Async Events API
Queues events for reliable background processing during high-traffic periods, returning a correlation id that can be polled for status. Uses token-based authentication via the `/v1/auth/token` endpoint.

### Antavo Customer API
Search, retrieve, and manage loyalty member profiles - including login, opt-in registration, password reset, verification, account merging, and active-customer counts - while maintaining member privacy.

### Antavo Display API
The primary headless API for building the member-facing loyalty experience - listing earn and spend activities, challenges, rewards, offers, coupons, transactions, wallet passes, quizzes, contests, prize wheels, and profiling flows, plus claiming and revoking rewards.

### Antavo Entities API
Generic CRUD surface for the foundational building blocks of a program - rewards, challenges, stores, products, transactions, and customer lists - addressed as entities under a module namespace, with list, create, retrieve, update, archive, and bulk submission.

### Antavo Rewards API
Manage the reward catalog and redemptions - create, list, retrieve, update, and archive rewards via the entities surface, and claim rewards. Legacy `/rewards` claim endpoints are superseded by the Display API.

### Antavo Coupons and Coupon Pools API
Query coupon usage independent of a customer and create or manage coupon pools - configuring value, expiration, and code patterns - with bulk import of codes and status/error reporting on the batch operations.

### Antavo Offers API
Submit a cart and retrieve applicable pre-purchase offers used for customer acquisition and engagement, and list a member's available offers through the Display surface.

### Antavo Points Preview API
Preview the loyalty points a transaction would earn, including bonus points assigned by the Workflows module, before the transaction is committed.

### Antavo Leaderboard API
Retrieve ranked lists of top customers with their scores for display in mobile apps, websites, and CRMs.

### Antavo Bulk Operations API
Batch processing for reward claims across many customers and for adding or removing customers from lists, each returning a batch id with status and error reporting endpoints.

### Antavo Clubs API
Create and administer member clubs and communities - templates, membership, invitations, applicants, bans, ownership, point adjustments and donations, history, and disbanding.

### Antavo Promotion Engine API
List and manage promotions and apply them at checkout - submit a cart to retrieve applicable promotions and finalize the checkout with the resulting discounts.

### Antavo Authentication API
Generate short-lived access tokens for credential clients configured in the Authentication Manager, used for token-based authentication such as the Async Events API.

## WebSocket Review

Antavo does **not** expose a documented public WebSocket API. Its own public interface is request/response REST over HTTPS. Event-driven behavior is delivered through incoming webhooks (external systems POST customer data that is registered as loyalty events), outbound Actions/webhooks (Antavo POSTs to a configured HTTP endpoint), and the poll-based Async Events API - none of which are a server-push WebSocket or SSE transport. See [review.yml](review.yml).

## Common Properties

- [GitHub Organization](https://github.com/antavo)
- [LinkedIn](https://www.linkedin.com/company/antavo)
- [Website](https://antavo.com)
- [Documentation](https://developers.antavo.com/docs/antavo-apis)
- [Plans](plans/antavo-plans-pricing.yml)
- [Rate Limits](rate-limits/antavo-rate-limits.yml)
- [Fin Ops](finops/antavo-finops.yml)
- [Blog](https://antavo.com/blog/)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
