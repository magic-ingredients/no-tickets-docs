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
status: not_started

In the Mintlify dashboard: Settings → Git → reconnect / reinstall
the Mintlify GitHub App against
`magic-ingredients/no-tickets-docs`, targeting branch `main`.
Confirm via `gh api repos/magic-ingredients/no-tickets-docs/hooks`
that a webhook now appears.

### 2. Trigger a redeploy to backfill stranded commits
status: not_started

Either push a trivial commit to `main` or use Mintlify's
dashboard "Redeploy" action. Verify the build picks up the latest
`api-reference/openapi.json` and the authored content under
`getting-started/`, `concepts/`, `integration-guides/`, etc.

### 3. Verify live matches local
status: not_started

Diff a few key pages between `docs.no-tickets.com` and
`http://localhost:3000` after the deploy finishes:

- `/introduction`
- `/api-reference/overview`
- One auto-generated endpoint page from the OpenAPI source

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
