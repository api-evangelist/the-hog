---
name: Build a prospect list with The Hog
description: Find target companies by firmographics, discover people at those accounts, and enrich the best contacts with verified emails and phones.
api: openapi/the-hog-openapi.json
operations: [searchCompanies, searchPeople, estimatePeopleSearch, submitEnrichment, getEnrichment, getOperation]
---

# Build a prospect list

Use this flow to turn an ICP into an enriched, contactable prospect list.

## Auth
Send both credentials on every request (from the dashboard Credentials page):
`X-Access-Key: ak_...` and `X-Secret-Key: sk_...`. Do not use `Authorization`.

## Steps
1. **Find target accounts** — `POST /api/v1/companies/search` (`searchCompanies`) with firmographic/technographic filters and buying signals. This is async: expect `202` with `operationId` + `pollUrl`, or a metered `200`. Pass an `Idempotency-Key` (UUID v4) so a retry never double-charges credits.
2. **Discover people** — `POST /api/v1/people/search` (`searchPeople`) scoped to the resolved accounts, filtering by role/seniority. Optionally call `POST /api/v1/people/search/estimate` (`estimatePeopleSearch`) first to preview credit cost and contact-enrichment risk.
3. **Poll** — for any `202`, poll `GET /api/operations/{id}` (`getOperation`) at 2–5s (search: 5–10s) until `status` is `succeeded`/`partial_success`. Read `result`. Read `targetAccountOutcome` / `companyMatchEvidence` to judge match quality.
4. **Enrich the winners** — `POST /api/enrichments` (`submitEnrichment`) with a JSON Schema for the fields you need. Returns `200` sync or `202` async; if async, poll `GET /api/enrichments/{id}` (`getEnrichment`) or the operation poll URL.

## Rules
- Idempotency-Key is org-scoped; generate a fresh UUID v4 per logical job, reuse it only for retries.
- Watch `metering.creditsCharged`; a `402` means insufficient credits (nothing was charged).
- Honor `Retry-After` on `429`; global limit is 600 req/min, polling 300 req/min.
- Errors follow the standard envelope (`statusCode/error/message/path/requestId/timestamp/errors`) — log `requestId` for support.
