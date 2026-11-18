# The roundtable-arsenal skills (AgentSkills.io format) as a kagent OCI skill source.
# kagent's Substrate pulls this image and mounts it read-only into the actor before the
# golden build, so the sandbox needs no network for skills (git sources are fetched
# inside the sandbox and crash the harness). Each skill is referenced from an
# AgentTemplate as source.oci = <this image>@sha256:<digest> with source.path = the
# skill's directory, e.g. vault/obsidian-conventions.
# Bump ARSENAL_COMMIT to publish newer skills.

ARG ARSENAL_COMMIT=75b1ade43e038779e1195740a965868ecbf4d87b

FROM alpine:3.22@sha256:49340b346798875b5592e5d420241f2f9cfba5f47e62ebb1446a1b34734e898f AS src
ARG ARSENAL_COMMIT
RUN apk add --no-cache git
WORKDIR /arsenal
RUN git init -q \
 && git remote add origin https://github.com/dapperdivers/roundtable-arsenal.git \
 && git fetch -q --depth 1 origin "${ARSENAL_COMMIT}" \
 && git checkout -q --detach FETCH_HEAD \
 && rm -rf .git .github

FROM scratch
LABEL org.opencontainers.image.source="https://github.com/dapperdivers/containers" \
      org.opencontainers.image.description="roundtable-arsenal skills as a kagent OCI skill source" \
      org.opencontainers.image.licenses="MIT"
COPY --from=src /arsenal /
