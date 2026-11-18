# containers

Container images built for the [dapper-cluster](https://github.com/dapperdivers/dapper-cluster)
homelab, published to `ghcr.io/dapperdivers/<name>`.

| Image | Source | Why we build it |
|---|---|---|
| `miniflux-mcp` | [tssujt/miniflux-mcp](https://github.com/tssujt/miniflux-mcp) at a pinned tag | The upstream image runs as root and is published from an unrelated Docker Hub account. Ours is built from source, non-root, on a digest-pinned distroless base. |

Each image lives in its own directory with a `Dockerfile` and a workflow in
`.github/workflows/`. Tags are the upstream version, never `latest`.
