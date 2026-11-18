# kagent's Claude Code harness image plus the CLI tools that agent skills expect
# (curl, jq, python3) and a baked Claude Code managed-settings policy.
# Used as Harness.spec.workload.image, referenced by sha256 digest.
#
# Contract with kagent (do not break):
#   - ENTRYPOINT, USER (65532:65532) and WORKDIR (/data) stay exactly as the base sets them.
#   - Nothing is written under /data, /tmp or /run/kagent: they are mounted over at runtime.
#   - Claude Code is NOT installed or upgraded here; the version is whatever the base ships.
#
# BASE must equal the claude harness image of the running kagent release (the `claude`
# entry of the controller's harnessImages). On every kagent upgrade: re-pin BASE, set
# KAGENT_VERSION to that release and reset REVISION to 1. For a rebuild on the same
# base (package or policy change) bump REVISION only. The tag is KAGENT_VERSION-rREVISION.

ARG KAGENT_VERSION=1.0.1
ARG REVISION=1
ARG BASE=ghcr.io/kagent-dev/kagent/claude-harness@sha256:111d28bc8552469b028d91ecd2352c9d44811aba07be12b1e6868db7948f904e

FROM ${BASE}
LABEL org.opencontainers.image.source="https://github.com/dapperdivers/containers" \
      org.opencontainers.image.description="kagent claude-harness plus curl, jq, python3 and a managed-settings policy"

USER root

# curl, jq and python3 are what skill scripts shell out to. py3-requests and py3-yaml are
# included because skill scripts commonly import them and nothing can be pip-installed
# into the root-owned system site-packages at runtime. py3-pip is there for scripts that
# call it (e.g. `pip list`). ca-certificates is already in the base.
RUN apk add --no-cache curl jq python3 py3-pip py3-requests py3-yaml \
 && install -d -o root -g root -m 0755 /etc/claude-code

# Claude Code managed settings: the highest-precedence settings file, which user and
# project settings cannot override. It denies the miniflux MCP server's destructive tools
# (Claude Code names MCP tools mcp__<server>__<tool>):
#   denied  - everything that deletes (feed, category, user, API key), flush_history,
#             credential and user management (create_api_key, get_api_keys, create_user,
#             update_user) and the bulk mark-as-read tools (all / category / feed), which
#             cannot be undone in one step.
#   allowed - all other read tools, plus curation that is individually reversible:
#             create_feed, update_feed, create_category, update_category,
#             update_entry_status, toggle_starred, save_entry and the refresh_* tools.
# Every denied tool is listed by full name; the delete_* glob is a second line of defence
# for delete tools the server may add later. See README.md for the reasoning.
# The directory is created above with mode 0755: letting COPY create it would give it the
# file's 0644 mode, and Claude Code (uid 65532) then cannot read the policy at all.
COPY --chown=root:root --chmod=0644 managed-settings.json /etc/claude-code/managed-settings.json

USER 65532:65532
