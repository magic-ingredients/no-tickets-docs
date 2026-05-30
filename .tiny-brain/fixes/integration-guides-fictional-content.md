---
id: integration-guides-fictional-content
type: fix
title: Integration guides reference a fictional GH Action, wrong token name, and a UI flow that doesn't exist
phase: development
status: not_started
severity: high
created: 2026-05-26T00:00:00.000Z
updated: 2026-05-26T00:00:00.000Z
reported: 2026-05-26T00:00:00.000Z
resolved: null
---

# Fix: Integration guides reference a fictional GH Action, wrong token name, and a UI flow that doesn't exist

## Problem

Four of the five `integration-guides/` pages document a
parallel-universe integration surface that doesn't match the real
product. A reader following these pages today would hit dead links,
404 hosts, wrong env-var names, and a dashboard flow that doesn't
exist. This is public-facing fiction on the docs site.

## Drift surface (verified by grep)

| File | push-action | api.no-tickets.com | NT_PUSH_TOKEN | Project settings UI |
|---|---|---|---|---|
| `github-actions.mdx` | 3× | 2× | 3× | 1× |
| `pr-workflows.mdx` | 1× | — | 1× | 1× (Phase mapping) |
| `generic-ci.mdx` | — | — | 6× | 1× |
| `agent-sdks.mdx` | — | — | 5× | — |

What each reference is wrong about:

- **`no-tickets/push-action@v1`** — the `no-tickets` GitHub org
  returns 404. The action repo doesn't exist. Referenced as if it
  were the canonical way to push from GH Actions.
- **`api.no-tickets.com`** — no service deployed there. Used as
  the default `api-url` and described as the action's outbound
  call destination.
- **`NT_PUSH_TOKEN`** — the real env var is `NO_TICKETS_TOKEN`
  per `magic-ingredients/no-tickets/docs/install.md`. Every
  example using `NT_PUSH_TOKEN` would silently fail authentication
  if copy-pasted.
- **"Project settings → Tokens"** — no such dashboard flow.
  Tokens are minted via the CLI: `no-tickets token add <project>
  <token>`.
- **"Project settings → Workflow → Phase mapping"** (in
  `pr-workflows.mdx`) — same shape of issue; needs verification
  against the real dashboard.

## What is real

- `app.no-tickets.com` — dashboard, HTTP 200.
- `docs.no-tickets.com` — this site, HTTP 200.
- `magic-ingredients/no-tickets/docs/install.md` — canonical
  binary-first install + GitHub Actions recipe (curl-install,
  `no-tickets publish`, `NO_TICKETS_TOKEN`). This is the source
  of truth to rewrite against.

## Tasks

### 1. Confirm the canonical contract from install.md
status: completed

Fetched `magic-ingredients/no-tickets/docs/install.md` (commit
HEAD at time of writing) and verified the strings below against
GitHub / live hosts. Canonical reference block follows — every
integration-guides rewrite must match it verbatim.

**Install (Linux / macOS, used in all CI recipes):**

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://get.no-tickets.com | sh
```

Installer drops `no-tickets` and `no-tickets-mcp` into
`~/.local/bin/`. CI runners need that on PATH for subsequent
steps to find the binary.

**GitHub Actions PATH step (mandatory — installer modifies
shell rc files but each Actions step runs in a fresh shell that
doesn't source them):**

```yaml
echo "$HOME/.local/bin" >> $GITHUB_PATH
```

**Generic shell PATH step (CircleCI, Bitbucket, Jenkins, Drone,
GitLab):**

```sh
export PATH="$HOME/.local/bin:$PATH"
```

**Env-var contract:**

- Variable name: **`NO_TICKETS_TOKEN`** (the integration guides
  currently use `NT_PUSH_TOKEN` — wrong).
- Value shape: `nt_push_…` (the raw token string).
- The CLI reads `NO_TICKETS_TOKEN` before consulting any local
  registry, so CI doesn't need a registry file.

**Token minting (workstation only, not a dashboard flow):**

```bash
no-tickets token add <project> <token>
```

There is no "Project settings → Tokens" UI — the integration
guides invent this. Tokens are minted by an authenticated
workstation user via the CLI; the raw value goes into the CI
secret store.

**Publish command shape:**

```bash
no-tickets publish \
  --project <project> \
  --type <event-type> \
  --data '<json>'
```

Example from install.md:

```bash
no-tickets publish \
  --project my-project \
  --type ai.task.completed.v1 \
  --data '{"taskId":"${{ github.run_id }}","outcome":"success"}'
```

**Host URLs:**

- `https://get.no-tickets.com` — binary distribution
  (installer). Real.
- `https://app.no-tickets.com` — dashboard. Real (HTTP 200).
- `https://docs.no-tickets.com` — this site. Real (HTTP 200).
- `https://api.no-tickets.com` — **does not exist** (HTTP 404).
  install.md does not mention an API URL at all; the binary
  handles host selection internally and exposes no `api-url`
  flag. The integration guides invent both the host and the
  flag.

**Verifying the install in CI:**

```yaml
- run: no-tickets --version
```

Catches a broken release at install time rather than at first
publish.

**Confirmed non-existence (404 via `gh api` / curl):**

- `no-tickets/push-action` — the org returns 404.
- `magic-ingredients/no-tickets-push-action` — does not exist.
- `api.no-tickets.com` — no service deployed.

**Strong signal for Task 2:** install.md documents zero GitHub
Actions wrapper — only the binary recipe. The canonical source
has already made the call (option **b** in Task 2). The
integration guides invented the action; nothing upstream
references it.

### 2. Decide: thin push-action wrapper, or drop the action entirely
status: completed

Decision: **(b) for now, with (a) tracked as a parallel fix.**

The integration guides will be rewritten to document only the
binary flow (curl install + `no-tickets publish`) so they match
the canonical contract in install.md and stop describing software
that doesn't exist. A separate fix will track building the
`no-tickets-push-action` wrapper; once that ships, the GitHub
Actions guide can be updated to mention the action alongside the
binary recipe. Docs follow shipping software, not the other way
around.

This means tasks 3–6 below remove all action references rather
than rewriting them around a wrapper.

### 3. Rewrite github-actions.mdx
status: not_started

Replace the push-action-based workflow with the binary recipe
from `install.md`. Remove the `api.no-tickets.com` references.
Use `NO_TICKETS_TOKEN` and the CLI token-minting flow.

### 4. Rewrite pr-workflows.mdx
status: not_started

Same shape as Task 3 for the PR-specific bits. Also verify
"Project settings → Workflow → Phase mapping" actually exists
before referencing it; if it doesn't, replace with the real
configuration surface (likely a config file or CLI command).

### 5. Update generic-ci.mdx
status: not_started

Smaller surface — only the token-name and dashboard-flow
references need fixing. Replace `NT_PUSH_TOKEN` → `NO_TICKETS_TOKEN`
across all six occurrences. Replace "Project settings → Tokens"
with `no-tickets token add <project> <token>`.

### 6. Update agent-sdks.mdx
status: not_started

Same as Task 5 for the five `NT_PUSH_TOKEN` references. Verify
that the MCP server config block accurately reflects the real
`no-tickets-mcp` env contract while in there.

### 6a. Update api-reference/overview.mdx
status: in_progress

Same class of drift on the API reference landing page. Already
removed the stale `<Warning>` block about the placeholder
openapi sync (it was lying — the sync works). Still to do on
this page:

- **Base URL** (line ~34): currently documented as
  `https://api.no-tickets.com`, which 404s. install.md doesn't
  document an API URL at all — the binary handles host
  selection internally. Either pin the real public host once
  someone confirms what it is, or drop the "Base URL" section
  entirely until the API is publicly deployed.
- **`NT_API_URL` env var** (line ~38): no such override exists
  in the binary. Same shape as the `NT_PUSH_TOKEN` invention.
- **Auth section** (line ~29): "Push tokens are issued per
  project from the dashboard" — same fictional UI flow. Real
  flow is `no-tickets token add <project> <token>` on a
  workstation (per the canonical reference block in Task 1).
- **Auth table** (line ~23-27): "Bearer" header form is
  plausible but should be confirmed against the actual service
  before pinning.

### 7. Verify on live after deploy
status: not_started

After the rewrite commits push to `main` and Mintlify rebuilds,
spot-check each rewritten page on `docs.no-tickets.com` and
confirm a copy-paste of any code block would succeed against the
real product.

## Out of scope

- Building the push-action wrapper itself (Task 2 option a). If
  that path is chosen, it lives in a new repo.
- Fixing `cli-reference/` or `mcp-tools/` placeholder content —
  tracked under the upstream documentation-site PRD in
  `no-tickets-service`.
- API-reference rendering — separate issue tracked in
  [[api-reference-missing-operation-ids]].

## Resolution

Set `status: completed` once:

1. All four affected files no longer reference `no-tickets/push-action`,
   `api.no-tickets.com`, `NT_PUSH_TOKEN`, or the "Project
   settings → Tokens / Phase mapping" UI strings (unless those
   surfaces have since been shipped — verify, don't assume).
2. Live `docs.no-tickets.com/integration-guides/*` pages match
   the updated source.
3. Pin the SHA of the rewrite commit(s) in the resolution block.
