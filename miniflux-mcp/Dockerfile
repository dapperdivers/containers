# Builds tssujt/miniflux-mcp from a pinned upstream tag into a minimal non-root image.
# The upstream Dockerfile runs as root on alpine:latest, and the image it points at is
# published from an unrelated Docker Hub account, so we build from source ourselves.
# Bump VERSION to upgrade.

# renovate: datasource=github-tags depName=tssujt/miniflux-mcp
ARG VERSION=v0.8.0

FROM golang:1.27-alpine@sha256:70bd7ddea8b6cbe5d6943cde279d6b67feb10cc924fb9100d1d8757c1575b280 AS build
ARG VERSION
RUN apk add --no-cache git
WORKDIR /src
RUN git clone --depth 1 --branch "${VERSION}" https://github.com/tssujt/miniflux-mcp.git .
RUN CGO_ENABLED=0 GOOS=linux go build -trimpath \
    -ldflags "-s -w -X main.Version=${VERSION}" -o /miniflux-mcp .

FROM gcr.io/distroless/static-debian12:nonroot@sha256:afa5c872c891853ca7fcf1f12c3edb23f7eeef36189728842dd51042ff57f7ab
LABEL org.opencontainers.image.source="https://github.com/dapperdivers/containers" \
      org.opencontainers.image.description="tssujt/miniflux-mcp, built from source, non-root" \
      org.opencontainers.image.licenses="MIT"
COPY --from=build /miniflux-mcp /miniflux-mcp
USER 65532:65532
EXPOSE 8080
ENTRYPOINT ["/miniflux-mcp"]
