# containers

Container images built for the [dapper-cluster](https://github.com/dapperdivers/dapper-cluster)
homelab, published to `ghcr.io/dapperdivers/<name>`.

| Image | Source | Why we build it |
|---|---|---|
| `miniflux-mcp` | [tssujt/miniflux-mcp](https://github.com/tssujt/miniflux-mcp) at a pinned tag | The upstream image runs as root and is published from an unrelated Docker Hub account. Ours is built from source, non-root, on a digest-pinned distroless base. |
| `claude-harness-tools` | kagent's `claude-harness` image at a pinned digest | The stock harness has no `curl`, `jq` or `python3`, which agent skills expect, and no way to ship a Claude Code policy. Ours adds both and changes nothing else. |
| `vault-mcp` | [MCPVault](https://www.npmjs.com/package/@bitbonsai/mcpvault) and [supergateway](https://www.npmjs.com/package/supergateway) from npm at pinned versions | MCPVault only speaks stdio, so supergateway serves it over HTTP. The cluster used to download both with `npx` at every pod start; this bakes them in, non-root, with no writable path needed. |

Each image lives in its own directory with a `Dockerfile` and a workflow in
`.github/workflows/`. Tags are the upstream version, never `latest`. `vault-mcp` packages
two upstream versions, so its tag is the build date and run number (`2026.11.24.1`).

## claude-harness-tools

kagent's Claude Code harness image with `curl`, `jq`, `python3`, `py3-pip`, `py3-requests`
and `py3-yaml` added, plus a Claude Code managed-settings file. It is meant for
`Harness.spec.workload.image` and must be referenced by `sha256` digest. The entrypoint,
user (`65532:65532`), working directory and Claude Code version are exactly the base's.

**Upgrade rule.** The base digest must be the `claude` harness image of the kagent release
that is running. On every kagent upgrade, re-pin `BASE`, set `KAGENT_VERSION` and reset
`REVISION` to `1` in the `Dockerfile`, then point the Harness at the new digest. For a
rebuild on the same base, bump `REVISION` only. Tags are `<kagent version>-r<revision>`,
e.g. `1.0.1-r1`.

**Managed-settings policy.** `/etc/claude-code/managed-settings.json` holds permission
`deny` rules that user and project settings cannot override. They remove the destructive
tools of the `miniflux` MCP server (Claude Code names MCP tools `mcp__<server>__<tool>`,
so the rules only apply while the server is registered under the name `miniflux`):

| | Tools | Why |
|---|---|---|
| Denied | `delete_feed`, `delete_category`, `delete_user`, `delete_api_key`, and the glob `delete_*` | Deletion cannot be undone. The glob covers delete tools added later. |
| Denied | `flush_history` | Irreversibly drops the read history. |
| Denied | `create_api_key`, `get_api_keys`, `create_user`, `update_user` | Credential and account management is not an agent's job. `update_user` is not exposed today and is listed defensively. |
| Denied | `mark_all_as_read`, `mark_category_as_read`, `mark_feed_as_read` | Bulk changes that cannot be undone in one step. |
| Allowed | every other read tool (`get_*`, `discover`, `export`, `fetch_*`, `healthcheck`) | Read-only. |
| Allowed | `create_feed`, `update_feed`, `create_category`, `update_category`, `update_entry_status`, `toggle_starred`, `save_entry`, `refresh_feed`, `refresh_category`, `refresh_all_feeds` | Curation that is individually reversible. |

When the MCP server gains tools, review them against this table: anything not denied is
allowed.
