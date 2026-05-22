---
id: docs-site-analytics
type: fix
title: Wire Mintlify analytics integration into docs.json
phase: development
status: not_started
severity: low
created: 2026-05-22T00:00:00.000Z
updated: 2026-05-22T00:00:00.000Z
reported: 2026-05-22T00:00:00.000Z
resolved: null
---

# Fix: Wire Mintlify analytics integration into docs.json

## Canonical PRD

This fix carves out the last unfinished sub-task of the docs-site
feature in the canonical PRD:

**PRD:** https://github.com/magic-ingredients/no-tickets-service/blob/main/docs/prd/no-tickets-team-dashboard/features/documentation-site.md
**Feature:** Documentation Site — Mintlify + Marketing Link-Out + In-App Help Button (Task 11 in that file's numbering, Task 23-16 in the dashboard PRD's flat task list).

Search and feedback are already enabled in `docs.json` (`search.prompt`
set; `feedback.thumbsRating` / `suggestEdits` / `raiseIssue` all true).
Analytics is the only remaining gap: `integrations` is currently
`null`. The canonical task is now tracked here; the corresponding
entry in the canonical PRD has been marked `superseded` with an
`implementedIn: no-tickets-docs` pointer.

## Problem

The docs site has no analytics wired up, so we have no visibility
into which sections users land on, what they bounce away from, or
how search queries trend. Mintlify ships native integrations for
PostHog, GA4, Plausible, Mixpanel, Fathom, Hotjar, Heap, and
LogRocket — picking whichever the marketing site uses and
declaring it in `docs.json` is a one-key change.

## Decision needed first

The marketing site (`packages/notickets-website` in the service
repo) does not currently use an analytics provider. Until that
choice is made, this fix is parked. The dependency runs one way:
the docs site adopts whatever the marketing site standardises on,
so the docs choice mirrors the broader product analytics decision
rather than driving it.

## Tasks

### 1. Pick the analytics provider on the marketing site
status: not_started

Coordinate with the marketing-site owner; the docs site adopts the
same provider for consistent attribution across the funnel
(marketing → docs → dashboard).

### 2. Wire the chosen provider into docs.json
status: not_started

Add `integrations.<provider>: { ... }` to `docs.json` per
Mintlify's integration docs. One-key change once the provider is
chosen.

**Files to modify:**
- `docs.json` — add the `integrations` block

### 3. Smoke-test that events fire
status: not_started

Open `docs.no-tickets.com`, navigate a couple of pages, and
verify events land in the chosen provider's dashboard.

## Out of scope

- Custom analytics endpoint (a Cloudflare Worker was originally
  proposed in the PRD; reframed during Task 1 — start with what
  Mintlify ships and only build custom if volume justifies it).
- Search and feedback configuration — already done.

## Resolution

Set `status: completed` and pin the `docs.json` commit SHA once
events are verified firing. Notify the service-repo PRD so the
canonical task's `superseded` pointer can be tightened with the
landing SHA.
