# no-tickets-docs

Source for [docs.no-tickets.com](https://docs.no-tickets.com) — built with
[Mintlify](https://mintlify.com).

This is the canonical home for:

- Getting started + install instructions
- The markdown format spec (epics, features, fixes, frontmatter, tasks)
- CLI reference (auto-generated from the [`no-tickets`](https://github.com/magic-ingredients/no-tickets) binary)
- REST API reference (auto-generated from `no-tickets-service`'s OpenAPI export)
- MCP tools reference
- Integration guides (GitHub Actions, generic CI, PR workflows, agent SDKs)
- Concepts (phases, push origins, entitlements, roles, epics)
- Troubleshooting and FAQ

## Why a separate repo

Docs span both products. The API reference is generated from
`no-tickets-service` (Hono+zod → OpenAPI). The CLI reference is generated from
the `no-tickets` Rust client. Putting docs in either repo would make the other a
second-class contributor that has to push artifacts over CI anyway. A separate
public repo gives both products a symmetric model and enables external typo /
clarity PRs.

Decision recorded in
[`no-tickets-service/docs/prd/no-tickets-team-dashboard/features/documentation-site.md`](https://github.com/magic-ingredients/no-tickets-service/blob/main/docs/prd/no-tickets-team-dashboard/features/documentation-site.md#decision-separate-public-sibling-repo).

## Local development

```bash
# One-time
npm i -g mint

# Every change
mint dev
```

The dev server runs on `http://localhost:3000` and hot-reloads on save.

`mint broken-links` checks for broken links across the navigation tree before
pushing.

## Deploy

Mintlify deploys this repo on every push to `main` (Git integration, configured
through the Mintlify dashboard). `docs.no-tickets.com` is the production
hostname.

Auto-generated artifacts (OpenAPI spec, CLI reference MDX) land in this repo
via the `.github/workflows/sync.yml` workflow on release-tag webhooks from the
two source repos — *not yet wired*; tracked as Tasks 2-4 of the
documentation-site feature.

## Sequencing

The public deploy is **gated** on the `event-repository-foundation` PRD landing
in `no-tickets-service`. Until then, content authored here describes a model
that is days from being deleted. The site builds locally (and may build into a
Mintlify preview), but the public DNS flip waits.

See `documentation-site.md` for the full sequencing rationale.

## Structure

```
docs.json              # Mintlify config (navigation, branding, search)
introduction.mdx       # Landing page
images/                # Logo + favicon
getting-started/       # Install, first push, dashboard tour
markdown-format/       # Format spec (overview + per-template)
cli-reference/         # Auto-synced from no-tickets binary
api-reference/         # Auto-synced from no-tickets-service OpenAPI
mcp-tools/             # Auto-synced from no-tickets-mcp tool registry
integration-guides/    # GitHub Actions, generic CI, PR, agent SDKs
concepts/              # Phases, push origins, entitlements, roles, epics
troubleshooting/       # Common errors
faq/                   # Pricing, data retention, privacy
snippets/              # Reusable MDX fragments
```

## Editing guidelines

- The CLI / API / MCP reference sections are **generated** — edits there get
  overwritten on next sync. Send PRs against the source repos instead.
- Prose sections (`markdown-format`, `concepts`, `integration-guides`,
  `troubleshooting`, `faq`) are hand-edited here. External PRs welcome.
- Cross-link liberally — `[label](/relative/path)` resolves within Mintlify;
  absolute `https://` links open in a new tab.
