---
id: api-reference-exclude-waitlist
type: fix
title: Exclude POST /v1/waitlist from the public API reference (internal-only endpoint)
phase: development
status: not_started
severity: medium
created: 2026-05-29T00:00:00.000Z
updated: 2026-05-29T00:00:00.000Z
reported: 2026-05-29T00:00:00.000Z
resolved: null
---

# Fix: Exclude POST /v1/waitlist from the public API reference

## Problem

`POST /v1/waitlist` currently appears in `api-reference/openapi.json`
and would surface as a public endpoint page on the live API
reference once the upstream `operationId: null` rendering bug
([[api-reference-missing-operation-ids]]) is fixed.

The waitlist endpoint is only consumed by the marketing site
(`packages/notickets-website`) for landing-page signups. It is
not part of the product API surface that customers / CLI users /
integrators are expected to call. Documenting it publicly would:

- Imply waitlist is a stable, supported integration point.
- Surface a route that has no value to API consumers.
- Lock in the schema as a public contract when it's really
  internal plumbing.

## Why this is an upstream fix

`api-reference/openapi.json` is committed in this repo via
Pattern B — `no-tickets-service` pushes it on each release. The
exclusion has to happen at the export step in `no-tickets-service`,
not by post-processing here. Patching in this repo would mask
the upstream shape and the next sync would silently put waitlist
back.

## Tasks

### 1. File the upstream issue against no-tickets-service
status: not_started

Open an issue / PR in `magic-ingredients/no-tickets-service`
asking to exclude `POST /v1/waitlist` from the public OpenAPI
export. Likely options at the route definition:

- Mark the route with a tag the export filter ignores
  (e.g. `tags: ["internal"]` + export skips internal-tagged
  routes).
- Add an `x-internal: true` extension on the operation +
  filter on export.
- Move the route to a separate Hono sub-app whose OpenAPI is
  not bundled into the public export.

The shape of the fix is the service's call; this fix just asks
for the outcome.

### 2. Verify the next openapi sync excludes /v1/waitlist
status: not_started

After the upstream change lands and a new
`chore(api-reference): sync openapi.json from no-tickets-service`
commit reaches `main` here:

```bash
jq '.paths | keys' api-reference/openapi.json
```

should no longer include `/v1/waitlist`. The API reference
sidebar on live should not list it either.

## Out of scope

- Hiding waitlist by post-processing `openapi.json` in this
  repo. Pattern B is "commit what the service exports" —
  filtering here would mask the upstream shape.
- The broader `operationId: null` rendering issue (tracked in
  [[api-reference-missing-operation-ids]]). The two fixes are
  independent: even with operationIds added, `/v1/waitlist`
  shouldn't render publicly.

## Resolution

Set `status: completed` once the openapi sync excludes
`/v1/waitlist` and live no longer shows it. Pin the sync commit
SHA.
