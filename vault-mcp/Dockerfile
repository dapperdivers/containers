# MCPVault (an Obsidian vault MCP server that only speaks stdio) with supergateway, which
# serves it as Streamable HTTP. Both are upstream npm packages, installed here at pinned
# versions so the pod does not download them with npx every time it starts.
#
# The entrypoint is supergateway; the caller supplies its arguments, for example
#   --stdio "mcpvault /vault --read-only" --outputTransport streamableHttp
#   --port 8000 --healthEndpoint /healthz
# and the API key in SUPERGATEWAY_API_KEY. Mount the vault at the path given to mcpvault.
# Runs as uid/gid 1000 (the owner of a Syncthing-managed vault) and writes nothing; npm's
# cache and update check are turned off so a read-only root filesystem needs no /tmp.
#
# Bump either version to upgrade.

FROM node:26.13.1-alpine@sha256:6bd9ab43c0ec0190ca7b2d193906cb8b7edf6e7f4afa43ddeccd6452710bffb5

# renovate: datasource=npm depName=@bitbonsai/mcpvault
ARG MCPVAULT_VERSION=0.18.4
# renovate: datasource=npm depName=supergateway
ARG SUPERGATEWAY_VERSION=4.2.2

LABEL org.opencontainers.image.source="https://github.com/dapperdivers/containers" \
      org.opencontainers.image.description="MCPVault served over Streamable HTTP by supergateway, non-root" \
      org.opencontainers.image.licenses="MIT"

ENV NODE_ENV=production \
    NPM_CONFIG_UPDATE_NOTIFIER=false \
    HOME=/tmp

RUN npm install --global --omit=dev --no-audit --no-fund \
        "@bitbonsai/mcpvault@${MCPVAULT_VERSION}" \
        "supergateway@${SUPERGATEWAY_VERSION}" \
 && npm cache clean --force \
 && rm -rf /root/.npm /tmp/*

USER 1000:1000
EXPOSE 8000
ENTRYPOINT ["supergateway"]
CMD ["--help"]
