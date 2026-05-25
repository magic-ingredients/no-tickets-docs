---
id: api-reference-missing-operation-ids
type: fix
title: OpenAPI export from no-tickets-service is missing operationId on every route
phase: development
status: not_started
severity: medium
created: 2026-05-25T00:00:00.000Z
updated: 2026-05-25T00:00:00.000Z
reported: 2026-05-25T00:00:00.000Z
resolved: null
---

# Fix: OpenAPI export from no-tickets-service is missing operationId on every route

## Problem

The live API reference at `docs.no-tickets.com/api-reference`
renders only **one** of the API's endpoints — `GET /health`,
slugged `system/liveness-probe`. The other five paths from
`api-reference/openapi.json` (7 operations) get no sidebar entry
and 404 on direct URL probes:

- `POST /v1/waitlist`
- `GET /v1/auth/me`
- `GET /v1/projects`
- `POST /v1/projects`
- `PATCH /v1/projects/{projectId}`
- `DELETE /v1/projects/{projectId}`
- `GET /v1/projects/{projectId}/feed`

Local and live serve the same `openapi.json` (6 paths, version
0.1.0), so this isn't a deploy / sync issue — it's a rendering
gap in Mintlify caused by upstream OpenAPI shape.

## Root cause

Every operation in the exported `openapi.json` has
`operationId: null`:

```bash
$ jq -r '.paths | to_entries[] | .key as $p | .value | to_entries[] | "\(.key | ascii_upcase) \($p)  operationId=\(.value.operationId)"' api-reference/openapi.json
GET /health  operationId=null
POST /v1/waitlist  operationId=null
GET /v1/auth/me  operationId=null
...
```

Mintlify uses `operationId` as the canonical page slug. When it's
missing, the renderer falls back inconsistently — `GET /health`
slugs to `system/liveness-probe` (tag + summary), but the other
ops are silently dropped.

The fix has to land in `no-tickets-service`, not here. Most
likely a missing `operationId: "..."` field on each Hono/zod
route definition that feeds the OpenAPI export.

## Why severity: high

This is the API reference. Discovering five out of six endpoints
get no documentation page is a credibility issue for the public
docs site, and it actively misleads readers into thinking the API
has only one endpoint.

## Tasks

### 1. File the upstream issue against no-tickets-service
status: not_started

Open an issue / PR in `magic-ingredients/no-tickets-service` with:

- The diagnosis above (jq output, list of affected paths).
- The required fix: add `operationId` to every OpenAPI operation
  on every route. Convention: kebab-case verb + resource
  (`list-projects`, `create-project`, `delete-project`,
  `get-project-feed`, `get-current-user`, `join-waitlist`).
- Note that the docs site auto-ingests on each release publish
  (Pattern B), so once the upstream fix lands and a new openapi
  sync commit reaches `main` here, the docs will rebuild.

### 2. Verify the next openapi sync renders all endpoints
status: not_started

After the upstream fix and the next `chore(api-reference): sync
openapi.json from no-tickets-service` commit:

- `jq -r '.paths | to_entries[] | .key as $p | .value | to_entries[] | "\(.key) operationId=\(.value.operationId)"' api-reference/openapi.json`
  should show non-null operationIds.
- `docs.no-tickets.com/api-reference/overview` sidebar should
  list all endpoints under their tag groups.
- Direct page loads (e.g. `/api-reference/endpoints/projects/list-projects`)
  should return HTTP 200.

## Out of scope

- Changes to the Mintlify rendering config in `docs.json`. The
  fallback behaviour is inconsistent but the correct upstream
  shape is to set `operationId` regardless.
- Working around in `no-tickets-docs` by post-processing the
  ingested `openapi.json`. Pattern B is "commit what the service
  exports" — patching here would mask the upstream bug.

## Resolution

Set `status: completed` once the verification in Task 2 passes.
Pin the openapi sync commit SHA that surfaces all endpoints.
