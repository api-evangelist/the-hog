---
name: Monitor social signals with The Hog
description: Create recurring monitors over keywords, profiles, companies, or posts across LinkedIn, X, Reddit, Instagram, and TikTok, then read detected events.
api: openapi/the-hog-openapi.json
operations: [createMonitor, runMonitorNow, listMonitorEvents, listMonitors, getMonitor, updateMonitor, deleteMonitor]
---

# Social listening with monitors

Use this to track competitors, communities, executives, and keywords over time.

## Auth
`X-Access-Key` + `X-Secret-Key` headers on every request.

## Steps
1. **Create a monitor** — `POST /api/v1/monitors` (`createMonitor`) with a `type` (e.g. `linkedin_keyword`, `x_profile`, `reddit_subreddit`, `tiktok_keyword`), a `config`, and a `cadence_minutes`.
2. **Run it now (optional)** — `POST /api/v1/monitors/{id}/run-now` (`runMonitorNow`) to trigger an immediate first pass instead of waiting for the cadence.
3. **Read events** — `GET /api/v1/monitors/{id}/events` (`listMonitorEvents`). Events are **pulled** (no webhook push); page forward with `limit` + `cursor` and follow `next_cursor`. Each event carries `event_type`, `event_json`, and `dedup_key`, and may link `canonical_person_id` / `canonical_company_id`.
4. **Manage** — `listMonitors`, `getMonitor`, `updateMonitor` (change config/cadence/status), `deleteMonitor` to remove.

## Rules
- There is no push delivery — poll `listMonitorEvents` on your own cadence and rely on `dedup_key` to skip duplicates.
- Respect the 600 req/min global limit; back off on `429`.
