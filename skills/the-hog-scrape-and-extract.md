---
name: Scrape and extract web data with The Hog
description: Search the web, scrape or crawl pages, and turn raw content into schema-guided structured JSON.
api: openapi/the-hog-openapi.json
operations: [searchWeb, scrapeWebPage, crawlWebSite, createWebScrapeJob, startDeepResearch, getOperation]
---

# Scrape and extract

Use this to pull content from the open web and return structured data.

## Auth
`X-Access-Key` + `X-Secret-Key` headers on every request.

## Steps
1. **Find URLs (optional)** — `POST /api/v1/platform/scrapers/web/search` (`searchWeb`) to discover relevant pages.
2. **Scrape a page** — `POST /api/v1/platform/scrapers/web/scrape` (`scrapeWebPage`) to return text, markdown, HTML, links, metadata, or schema-guided JSON for one URL.
3. **Crawl / queue for scale** — `POST /api/v1/platform/scrapers/web/crawl` (`crawlWebSite`) for a site, or `POST /api/v1/platform/scrapers/web/scrape/jobs` (`createWebScrapeJob`) for a deep async job (poll `getOperation`).
4. **Extract structure** — pass a JSON Schema to the scrape call, or hand the scraped content to `POST /api/deep-research` (`startDeepResearch`) for LLM extraction, then poll `GET /api/operations/{id}` (`getOperation`).

## Rules
- Scrape calls are credit-metered; check `metering.creditsCharged`. A `402` means insufficient credits.
- Use `Idempotency-Key` on the async scrape-job and deep-research calls.
