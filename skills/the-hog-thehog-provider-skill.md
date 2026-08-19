---
name: Thehog
description: Use when building GTM intelligence workflows: searching for target companies by firmographics and signals, discovering and enriching contacts, running LLM-powered research, monitoring social mentions, or scraping web and social data. Agents should reach for this skill when users need to find accounts, build prospect lists, enrich contact data, research companies, or set up recurring monitoring.
metadata:
    mintlify-proj: thehog
    version: "1.0"
---

# The Hog API Skill

## Product summary

The Hog is a REST API for go-to-market teams that need fast, accurate data about target accounts and contacts. With a single API, you can search companies by firmographics and technographics, discover and enrich people, run LLM-powered deep research, monitor social mentions, and scrape web and social platforms. All requests go to `https://developer.thehog.ai/api` with authentication via `X-Access-Key` and `X-Secret-Key` headers. The API uses async operations (202 responses) for long-running tasks — you submit a job and poll `GET /api/operations/:id` for results. Key endpoints: `POST /api/v1/companies/search`, `POST /api/v1/people/search`, `POST /api/enrichments`, `POST /api/deep-research`, `POST /api/v1/monitors`, and `POST /api/v1/search`. See the full API reference at https://docs.thehog.ai/api-reference.

## When to use

Reach for The Hog API when:
- **Building prospect lists**: Search target companies by industry, employee count, revenue, tech stack, hiring signals, or funding activity, then drill into contacts at those accounts.
- **Enriching contacts**: Get verified emails and phone numbers for prospects from LinkedIn URLs, email addresses, X handles, or GitHub usernames.
- **Running research**: Submit open-ended research questions with a JSON Schema and get back structured data extracted from web sources.
- **Monitoring mentions**: Set up recurring monitors to track keywords, profiles, or posts across LinkedIn, X, Reddit, TikTok, and the web.
- **Scraping social and web**: Fetch public profile data, posts, comments, followers, or web page content from LinkedIn, Instagram, TikTok, Facebook, YouTube, or arbitrary websites.
- **Estimating costs**: Use estimate endpoints to preview credit consumption before submitting expensive operations.

Do not use The Hog API for: dashboard-only operations, account management, authentication setup, or operations that don't involve external data fetching.

## Quick reference

### Authentication headers
```
X-Access-Key: ak_xxxxxxxxxxxxxxxx
X-Secret-Key: sk_xxxxxxxxxxxxxxxx
```
Retrieve from the Credentials page in your dashboard. Never expose in client-side code.

### Core endpoints
| Endpoint | Method | Pattern | Use for |
|----------|--------|---------|---------|
| `/api/v1/companies/search` | POST | Async (202) | Find target accounts by firmographics, technographics, signals |
| `/api/v1/people/search` | POST | Async (202) | Find contacts by role, company, location, natural language |
| `/api/enrichments` | POST | Sync (200) or Async (202) | Get verified emails, phone, signals for contacts |
| `/api/deep-research` | POST | Async (202) | Run LLM-powered research with JSON Schema output |
| `/api/v1/monitors` | POST | Sync (200) | Create recurring monitors for keywords, profiles, posts |
| `/api/v1/search` | POST | Async (202) | One-off searches across web and social platforms |
| `/api/operations/:id` | GET | Sync (200) | Poll async operation status and results |

### Company search filters
| Filter | Type | Example |
|--------|------|---------|
| `query` | string | `"Salesforce"` or `"Software companies using Salesforce"` |
| `filters.industries` | array | `["Software", "FinTech"]` |
| `filters.employeeCount.min/max` | number | `50` / `500` |
| `filters.locations` | array | `["United States", "Canada"]` |
| `filters.company.domains` | array | `["acme.com"]` |
| `filters.signals` | array | `["hiring", "funding"]` |
| `limit` | number | 1–100, default 25 |

### People search filters
| Filter | Type | Example |
|--------|------|---------|
| `query` | string | `"VP of Sales at Series B SaaS"` |
| `filters.titles` | array | `["VP Sales", "Head of Revenue"]` |
| `filters.locations` | array | `["New York", "San Francisco"]` |
| `filters.company.domains` | array | `["salesforce.com"]` |
| `includeContacts` | boolean | `true` to enrich during search |
| `contactFields` | array | `["email", "phone"]` |
| `limit` | number | 1–100, default 25 |

### Enrichment identifiers
| Field | Type | Example |
|-------|------|---------|
| `linkedin_url` | string | `https://www.linkedin.com/in/jane-doe` |
| `email` | string | `jane@example.com` |
| `x_handle` | string | `jane_sales` |
| `github_username` | string | `janedoe` |

### Operation status values
| Status | Meaning | Next step |
|--------|---------|-----------|
| `queued` | Waiting to start | Keep polling |
| `processing` | In progress (check `progress` field) | Keep polling |
| `succeeded` | Complete — read `result` | Extract data from result |
| `failed` | Error occurred — read `error` | Check error message, retry if appropriate |
| `partial_success` | Some items succeeded | Check both `result` and `error` |

### Rate limits
- **Global**: 600 requests/minute per organization-and-user
- **Polling**: 300 requests/minute per organization-and-user (separate bucket)
- **Recommended polling intervals**: 2–5 seconds for enrichment/search, 10–30 seconds for deep research

## Decision guidance

### When to use sync vs. async enrichment

| Scenario | Use | Pattern |
|----------|-----|---------|
| Single LinkedIn contact, email/phone only | Single `identifier` | Returns 200 or 202 |
| Multiple contacts or batch enrichment | `identifiers` array (up to 100) | Always returns 202 |
| Need signals or deep enrichment | Include `signals` field | Always returns 202 |
| Want to cap credit spend | Use `maxCredits` on people search | Fails with 402 if over budget |

### When to use company search vs. people search

| Goal | Use | Pattern |
|------|-----|---------|
| Build target account list | Company search first | Filter by firmographics, technographics, signals |
| Find contacts at known accounts | People search with `filters.company` | Scope to specific domains or names |
| Global contact discovery | People search without company filter | Broader but less focused results |
| Validate ICP before outreach | Company search + people search | Two-step workflow ensures quality |

### When to use monitors vs. one-off search

| Use case | Use | Cadence |
|----------|-----|---------|
| Track hot topics over time | Monitor with `x_keyword` type | Recurring (hourly, daily, weekly) |
| Watch competitor activity | Monitor with `x_keyword` type | Recurring |
| Track executive posts | Monitor with `x_profile` type | Recurring |
| Test query before monitoring | One-off search with `POST /api/v1/search` | Single run |
| Build social inbox | Monitor + poll events endpoint | Recurring with deduplication |

### When to use deep research vs. web scrape

| Need | Use | Output |
|------|-----|--------|
| Structured data matching schema | Deep research | JSON conforming to your schema |
| Raw page HTML or text | Web scrape | HTML, markdown, or extracted text |
| LLM synthesis from multiple sources | Deep research | Synthesized structured result |
| Single page content | Web scrape | Direct page content |

## Workflow

### Build a prospect list (company-first approach)

1. **Search for target companies**
   - Call `POST /api/v1/companies/search` with `query` and `filters` (industry, employee count, signals)
   - Receive 202 with `operationId`
   - Poll `GET /api/operations/:id` every 2–5 seconds until `status` is `succeeded`
   - Extract company domains or names from `result.data`

2. **Find contacts at those companies**
   - Call `POST /api/v1/people/search` with `query` (e.g., "VP of Sales") and `filters.company.domains` from step 1
   - Receive 202 with `operationId`
   - Poll `GET /api/operations/:id` until `status` is `succeeded`
   - Extract person IDs or LinkedIn URLs from `result.data`

3. **Enrich selected contacts**
   - Call `POST /api/enrichments` with `identifier` (LinkedIn URL or email) and `fields: ["contact.email", "contact.phone"]`
   - If single contact and email/phone only: may return 200 (sync) with data immediately
   - If batch or signals: returns 202 with `operationId`
   - Poll `GET /api/operations/:id` until `status` is `succeeded`
   - Extract verified emails and phone numbers from `result.data.contact`

4. **Verify and export**
   - Check `emailStatus` and `phoneStatus` fields for verification confidence
   - Export to CRM or outreach tool
   - Track which contacts were enriched synchronously vs. asynchronously

### Run deep research on a company

1. **Define your research question and schema**
   - Write a focused `prompt` (e.g., "Research Acme Corp. Find products, customers, recent funding, key executives")
   - Create a JSON Schema with required fields (e.g., `companyName`, `mainProducts`, `recentFunding`)

2. **Submit the research job**
   - Call `POST /api/deep-research` with `prompt`, `schema`, and optional `urls` (seed URLs)
   - Include `Idempotency-Key` header to prevent duplicate charges on retry
   - Receive 202 with `operationId`

3. **Poll for results**
   - Poll `GET /api/operations/:id` every 10–30 seconds (deep research is slower)
   - Check `progress` field to track completion percentage
   - When `status` is `succeeded`, read `result` — it will conform to your schema

4. **Consume the structured result**
   - Map result fields directly into your data pipeline or CRM
   - Reuse the same `Idempotency-Key` if you need to retry — you won't be charged twice

### Set up a social listening monitor

1. **Create the monitor**
   - Call `POST /api/v1/monitors` with `type` (e.g., `x_keyword` or `x_profile`), `config` (query or username), `cadence_minutes`, and `max_results`
   - Receive 200 with monitor ID

2. **Run the monitor immediately (optional)**
   - Call `POST /api/v1/monitors/:id/run-now` to trigger a run
   - Receive 202 with operation ID
   - Poll `GET /api/operations/:id` until complete

3. **Poll for new events**
   - Call `GET /api/v1/monitors/:id/events?since=<iso-timestamp>&limit=50`
   - Store the `detected_at` timestamp from the last successful poll
   - Pass it as `since` on the next poll to only fetch new items

4. **Process and route**
   - Extract `event_json.url`, `event_json.text`, `event_json.author_username`
   - Deduplicate by post URL or source-native ID
   - Score for actionability (buying intent, competitor signal, etc.)
   - Route to Slack, CRM, or task queue

## Common gotchas

- **Forgetting to poll async operations**: Company search, people search, deep research, and batch enrichment all return 202 immediately. You must poll `GET /api/operations/:id` to get results. Polling is free.

- **Polling too aggressively**: The polling endpoint has a separate 300 req/min limit. Poll at 2–5 second intervals for fast jobs, 10–30 seconds for deep research. Aggressive polling triggers 429 errors and blocks other requests.

- **Not using company-first workflow**: Searching for people globally without company filters produces noisy results. Always search companies first, then scope people search to those accounts.

- **Exceeding credit budget**: Company search, enrichment, and deep research charge credits. Use `POST /api/v1/people/search/estimate` to preview costs before running expensive operations. Set `maxCredits` to cap spend.

- **Batch enrichment returning 202 when expecting 200**: Single LinkedIn contact requests for email/phone only can return 200 (sync). Batches, signals, or other fields always return 202 (async). Check the response status code.

- **Idempotency key not preventing duplicate charges**: Idempotency keys only work on `POST` requests within the deduplication window. Use deterministic keys (e.g., `research-{company}-{month}`) so retries are safe.

- **Missing X-Request-Id for support**: Every error response includes `requestId` in the body and `X-Request-Id` in headers. Save this when contacting support — it maps directly to logs.

- **Validation errors with unclear messages**: 400 responses include an `errors` array with `property` and `message` fields. Fix every entry in the array before retrying.

- **Assuming all fields are present in results**: Some fields (like `phone` or `signals`) may be unavailable for a contact. Check `emailStatus` and `phoneStatus` fields to understand coverage.

- **Not handling partial_success status**: Some operations return `partial_success` when some items succeed and others fail. Check both `result` and `error` fields.

## Verification checklist

Before submitting work with The Hog API:

- [ ] **Authentication**: Verified API key and secret are correct and not exposed in logs or code
- [ ] **Async operations**: Confirmed all 202 responses are being polled until `status` is `succeeded` or `failed`
- [ ] **Polling interval**: Using 2–5 second intervals for search/enrichment, 10–30 seconds for deep research
- [ ] **Credit budget**: Ran estimate endpoint or checked `maxCredits` before submitting expensive operations
- [ ] **Company-first workflow**: For prospect lists, confirmed company search precedes people search
- [ ] **Idempotency keys**: Added `Idempotency-Key` header to deep research and batch enrichment requests
- [ ] **Error handling**: Checking `statusCode` and `errors` array on 400 responses; handling 402 (insufficient credits) and 429 (rate limit)
- [ ] **Data validation**: Confirmed required fields are present in results (e.g., `emailStatus` before using email)
- [ ] **Deduplication**: For monitors, storing post URL or source-native ID to avoid duplicate processing
- [ ] **Request ID logging**: Saving `X-Request-Id` header for debugging and support tickets

## Resources

- **Full API navigation**: https://docs.thehog.ai/llms.txt — comprehensive page-by-page reference for all endpoints and guides
- **API Reference**: https://docs.thehog.ai/api-reference — overview of all endpoint categories
- **Quickstart**: https://docs.thehog.ai/quickstart — step-by-step walkthrough of company search, people search, enrichment, and polling
- **Core Concepts**: https://docs.thehog.ai/concepts/company-first-search — company-first search philosophy and typical workflow

---

> For additional documentation and navigation, see: https://docs.thehog.ai/llms.txt