# makefun-github-core-clone

A standalone GitHub-compatible git forge in a single Python file
(`main.py`, standard library only — no git binary, no packages, no
outside service). One process serves four surfaces from its own
in-memory store:

- **git smart HTTP** — protocol v2 `upload-pack` (ls-refs, fetch,
  shallow/deepen) and v0 `receive-pack` (report-status, side-band-64k,
  fast-forward rules, mixed push reports), with a from-scratch pack
  reader/writer (plain, OFS_DELTA, REF_DELTA) and sideband framing.
- **REST v3 API** (`/api/v3`) — repositories, branches, refs, commits,
  contents, labels, issues (+ comments) and pull requests (create,
  files, squash merge with three-way conflict detection).
- **GraphQL v4** (`/api/graphql`) — hand-written parser + executor for
  the repository / issue / pull-request queries and mutations `gh` and
  `github-mcp-server` send, including introspection (`__type`),
  cursor pagination, global ids and batched operations.
- **MCP** (`/mcp`) — modern `server/discover` era (2026-07-28) with an
  `initialize` fallback; six tools: `create_repo`, `get_repo`,
  `create_issue`, `list_issues`, `create_pull_request`,
  `merge_pull_request`.

## Running

```sh
python3 main.py
```

Without `MAKEFUN_ENV_URL` it serves forever on `FORGE_PORT` (default:
an ephemeral port, printed to stderr). With `MAKEFUN_ENV_URL` set it
binds an ephemeral port, registers it with the suite environment, and
exits when the environment reports done. Authentication is
`Authorization: Bearer $MAKEFUN_SHOP_TOKEN`; when that variable is
unset any non-empty bearer token is accepted (local development).

The authenticated viewer is `adaiversen`. All timestamps are fixed
(`2025-01-01T00:00:00Z`) and ids are assigned in creation order, so
two runs on the same instance leave the same state export.

## Local conformance

```sh
MAKEFUN_SHOP_TOKEN=test-token FORGE_PORT=18080 python3 main.py &
node --experimental-strip-types <kit>/sdk/run.ts \
  --url http://127.0.0.1:18080 --token test-token --perf
```

Built for the make.fun `github-core-clone-01` bounty.
