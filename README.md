# Antavo (antavo)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
