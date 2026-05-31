---
id: docs-conflate-tiny-brain-and-no-tickets
type: fix
title: Docs site conflates tiny-brain's spec format with no-tickets' product surface
phase: development
status: not_started
severity: high
created: 2026-05-31T00:00:00.000Z
updated: 2026-05-31T00:00:00.000Z
reported: 2026-05-31T00:00:00.000Z
resolved: null
---

# Fix: Docs site conflates tiny-brain's spec format with no-tickets' product surface

## Problem

Multiple sections of this docs site present **tiny-brain's
behaviour as if it were no-tickets' product surface**. Per the
canonical framing on
[`no-tickets.com`](https://no-tickets.com) — and the FAQ at
[/faq/overview](https://docs.no-tickets.com/faq/overview) — the
two products are distinct:

- **tiny-brain** (open-source CLI, `tiny-brain.com`) lives in the
  repo, maintains `.tiny-brain/`, enforces TDD, runs reviews, and
  pushes structured state to no-tickets.
- **no-tickets** is the event-published-to backend + dashboard.
  Its public CLI (`no-tickets publish`) and REST API accept
  events. It does not natively scan a directory format.

The docs claim otherwise in three places.

### 1. markdown-format/ section

`markdown-format/overview.mdx` opens with:

> no-tickets stores work as plain markdown files in your
> repository. There is no database to sync, no GUI to drag
> through — the files are the spec. The CLI, the dashboard, and
> any LLM tool all read and write the same format.

This is the framing of an earlier vision that didn't survive
the pivot to event publishing. The format described
(`.tiny-brain/<epic>/{epic,feature,fix}.md`) IS real, but it
belongs to tiny-brain, not no-tickets. no-tickets accepts
events through the publish API; the markdown format is
tiny-brain's preferred way to generate those events.

Affects: `markdown-format/{overview,epics,features,fixes,frontmatter,task-syntax}.mdx`.

### 2. concepts/ section

`concepts/overview.mdx` says no-tickets "describes work in three
nouns — epics, features, and fixes — and one verb: push." The
nouns are real domain concepts; the "push" framing as the only
verb glosses over the event-publish model. `concepts/epics.mdx`
and `concepts/push-origins.mdx` deepen the conflation by
talking about scanning the `.tiny-brain/` directory.

Affects: `concepts/{overview,phases,push-origins,entitlements,roles,epics}.mdx`.

### 3. integration-guides/ section

The integration guides describe "push your `.tiny-brain/`
directory state on every CI run" — there is no `no-tickets push`
command that walks `.tiny-brain/`. The real CLI surface is
event publishing (`no-tickets publish --type --data`).

This is already partially tracked under
[[integration-guides-fictional-content]] (tasks 3-6 cover the
GH Action / env-var / token-UI drift), but the underlying
framing — "push directory state" vs "publish events" — needs
to be resolved before those rewrites can land coherently.

Affects: `integration-guides/{overview,github-actions,generic-ci,pr-workflows,agent-sdks}.mdx`.

## Why severity: high

The docs site is the canonical public reference for both
product surfaces. Today readers land on
`/markdown-format/overview` and learn a model that doesn't
match what the binary does. Engineers who copy-paste the
"directory push" framing into CI will hit blank failures because
no such command exists. The drift is foundational, not
cosmetic.

## Decision needed

The structural fix has three viable shapes. The right call
depends on the product owner's view on whether the
markdown-format spec should be hosted here at all.

### Option A: Move markdown-format to tiny-brain.com

Strongest separation. `docs.no-tickets.com` documents the
backend / API / dashboard; `tiny-brain.com` documents the
local CLI and its spec format. Cross-link prominently from
the no-tickets intro: "Most users generate state via
tiny-brain — see tiny-brain.com for its docs."

Cost: a section migration + tiny-brain.com needs to exist
as a real docs site (today it's a one-page marketing site).

### Option B: Keep markdown-format here, reframe as "the tiny-brain spec"

Treat the markdown-format section as documentation of
tiny-brain's behaviour, hosted on no-tickets' docs site as a
convenience. The opener becomes: "tiny-brain — the recommended
companion CLI — maintains state in `.tiny-brain/`. This is its
spec." All sentences claiming no-tickets reads the format get
rewritten.

Cost: rewrite ~6 pages of framing copy, no migration. Risk:
docs.no-tickets.com partially documents another product.

### Option C: Delete markdown-format from the docs site

Strip the section entirely. Replace with a single page that
says "no-tickets accepts events at its API — see the API
reference. The recommended way to generate events is
tiny-brain (link)." Same for the directory-push framing in
concepts/ and integration-guides/.

Cost: smallest surface to maintain. Loses real content that
exists nowhere else.

## Tasks

### 1. Pick a structural direction (A, B, or C)
status: not_started

Product call — needs the owner of the no-tickets / tiny-brain
docs strategy. Record the decision here before any rewrite
work starts. The other tasks below assume a direction has been
picked.

### 2. Rewrite markdown-format/ to match the chosen direction
status: not_started

Depending on direction:

- **A**: open PR against the tiny-brain repo / docs site
  moving the section. In this repo, replace the section with
  a single "the spec lives at tiny-brain.com" page.
- **B**: rewrite all 6 markdown-format pages to frame them as
  "the tiny-brain spec." Update introduction.mdx and the FAQ
  to match.
- **C**: delete the section. Update docs.json nav. Add a
  pointer page or update the API reference intro to mention
  tiny-brain as the recommended event generator.

### 3. Rewrite concepts/ to remove the directory-push framing
status: not_started

`concepts/overview.mdx`, `concepts/epics.mdx`,
`concepts/push-origins.mdx`. Same direction as Task 2. The
domain nouns (epic, feature, fix, phase, role, entitlement)
stay; the verb model ("push directory state") gets replaced
with the real event-publish model.

### 4. Unblock integration-guides rewrites
status: not_started

Once Tasks 2-3 land, the integration-guides rewrites under
[[integration-guides-fictional-content]] (tasks 3-6) can
proceed with consistent framing. Without this fix landing
first, those rewrites would either perpetuate the directory-
push framing or contradict the rest of the docs.

### 5. Update introduction.mdx
status: not_started

The intro currently hedges: "Specs live as markdown in your
repo. Agents push events as they work." Pick one model — the
event-publish model is canonical. Reframe the intro so a new
reader gets the right mental model on the first sentence.

## Out of scope

- The pure `.notickets/` → `.tiny-brain/` rename has already
  landed (see commit 6059ba4). This fix is about the deeper
  framing, not the directory name.
- The MCP tools section — that's blocked on the upstream MCP
  sync pipeline, separate concern.
- The CLI/API reference content — those are auto-generated
  from real sources and accurately reflect what ships.

## Resolution

Set `status: completed` once Tasks 1-5 are all done and
`docs.no-tickets.com` consistently frames no-tickets as the
event-published-to backend, with tiny-brain (and its spec
format) treated as a separate companion product. Pin the SHA
of the structural-rewrite commit(s).
