---
name: Run structured deep research with The Hog
description: Submit an LLM-powered research prompt plus a JSON Schema and get back structured, schema-conforming data extracted from web sources.
api: openapi/the-hog-openapi.json
operations: [startDeepResearch, getOperation]
---

# Structured deep research

Use this to answer an open research question and receive typed JSON instead of prose.

## Auth
`X-Access-Key` + `X-Secret-Key` headers on every request.

## Steps
1. **Start the job** — `POST /api/deep-research` (`startDeepResearch`) with a `prompt` and a JSON `schema` describing the exact shape you want back. Include an `Idempotency-Key` (UUID v4). Deep research is always async — expect `202` with `operationId` + `pollUrl`.
2. **Poll patiently** — `GET /api/operations/{id}` (`getOperation`) at **10–30s** intervals; deep research routinely takes 30s to several minutes. Stop when `status` is `succeeded` (or `partial_success`).
3. **Read the result** — `result` conforms to the JSON Schema you supplied. On `failed`, read `error.message`.

## Rules
- Poll slowly — aggressive polling trips the 300 req/min poll bucket (`429` + `Retry-After`).
- Retry a dropped `202` with the **same** Idempotency-Key to avoid a duplicate (double-credit) job.
- A `402` before any work means insufficient credits; nothing was charged.
