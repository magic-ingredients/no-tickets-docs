---
id: docs-site-deploy-webhook
type: fix
title: Reconnect Mintlify GitHub App so docs.no-tickets.com auto-deploys
phase: development
status: completed
severity: medium
created: 2026-05-25T00:00:00.000Z
updated: 2026-05-25T00:00:00.000Z
reported: 2026-05-25T00:00:00.000Z
resolved: 2026-05-25T11:00:00.000Z
resolution:
  rootCause: Mintlify project was originally hosted via the web editor with no GitHub App installed, so pushes to main never triggered a deploy and the live site drifted behind the repo.
  fix:
    - Connected the magic-ingredients/no-tickets-docs repo from the Mintlify dashboard (replacing the hosted-editor source).
    - First post-connect deploy backfilled all stranded commits from main.
    - Verified live pages match local for /introduction, /api-reference/overview, and a generated endpoint page.
  filesModified: []
archived: true
---

# Fix: Reconnect Mintlify GitHub App so docs.no-tickets.com auto-deploys

## Problem

The live docs site at `docs.no-tickets.com` is frozen at the
bootstrap scaffold and does not reflect any content authored after
the initial Mintlify project connection. Local `mint dev` reads the
filesystem and shows everything correctly, which is why local looks
materially better than live.

Diagnostic that confirmed the cause:

```
gh api repos/magic-ingredients/no-tickets-docs/hooks  →  []
ls .github/workflows                                  →  (none)
```

No webhook, no CI — nothing wired up to trigger a Mintlify deploy on
push to `main`. Mintlify's auto-deploy works via a GitHub App that
registers a push webhook on the repo; that App is not currently
installed on this repo (or was installed and later removed /
revoked).

## Commits stranded behind the missing webhook

At time of writing, none of these have reached the live site:

- `5aaca66` feat: author canonical docs content and wire OpenAPI ingestion
- `c4f9ac2` refactor(docs): fix factual drift across new docs pages
- `9689fea` feat(api-reference): switch to Pattern B — commit openapi.json from no-tickets-service
- `ab8567c` refactor(api-reference): tighten Pattern B follow-up details
- `98b13bd` docs(fixes): track docs-site analytics integration
- `d63850e` fix(api-reference): stub /health operation in placeholder OpenAPI
- `2d6fb54` chore(api-reference): sync openapi.json from no-tickets-service v0.1.0

## Tasks

### 1. Reconnect the Mintlify GitHub App on this repo
status: completed
commitSha: a3e0c68

Reconnected via the Mintlify dashboard. Note: `gh api repos/.../hooks`
still returns `[]` after reconnect — Mintlify's GitHub App uses
App-level event subscriptions rather than classic per-repo
webhooks, so the empty hooks list is expected and is not a sign of
broken integration. Deploys on push to `main` are firing.

### 2. Trigger a redeploy to backfill stranded commits
status: completed
commitSha: a3e0c68

The push of `a3e0c68` triggered an automatic Mintlify rebuild.
Live `api-reference/openapi.json` now matches local (6 paths,
version 0.1.0) and authored content under `getting-started/`,
`concepts/`, `integration-guides/`, etc. is live.

### 3. Verify live matches local
status: completed
commitSha: a3e0c68

Verified by curl against `docs.no-tickets.com` (local `mint dev`
wasn't running, but the deployed `openapi.json` and rendered
content match the repo's source files):

- `/introduction` returns 200 with the full authored nav.
- `/api-reference/overview` returns 200.
- `/api-reference/endpoints/system/liveness-probe` returns 200.

Caveat: only 1 of 8 operations in `openapi.json` renders an
endpoint page. This is an upstream OpenAPI shape bug, not a
deploy bug — tracked separately in
[[api-reference-missing-operation-ids]].

## Out of scope

- Analytics wiring — tracked separately in [[docs-site-analytics]].
- Switching deploy provider away from Mintlify's hosted build.
- Adding a GitHub Actions workflow as a backup deploy path —
  consider only if the Mintlify App proves unreliable after
  reconnect.

## Resolution

Set `status: completed` once:

1. `gh api repos/.../hooks` shows the Mintlify webhook.
2. A push to `main` triggers a successful Mintlify build.
3. Live and local render the same content for the three verification
   pages above.
