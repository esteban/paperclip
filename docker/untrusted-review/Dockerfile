# Pin to deterministic major version — floating `lts` tag causes non-reproducible builds
# and supply-chain risk from unreviewed upstream updates.
FROM node:22-trixie-slim

LABEL org.opencontainers.image.title="paperclip-untrusted-review" \
      org.opencontainers.image.description="Sandboxed reviewer — runs untrusted AI agents against submitted PRs" \
      security.pinned-base="node:22-trixie-slim" \
      security.pinned-packages="@anthropic-ai/claude-code@1.0.33,@openai/codex@0.1.2505302106"

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    bash \
    ca-certificates \
    curl \
    fd-find \
    gh \
    git \
    jq \
    less \
    openssh-client \
    procps \
    ripgrep \
  && rm -rf /var/lib/apt/lists/*

RUN ln -sf /usr/bin/fdfind /usr/local/bin/fd

RUN corepack enable \
  # Versions pinned to match main Dockerfile — @latest is a supply-chain attack surface.
  # --ignore-scripts prevents postinstall hooks from executing arbitrary code in this
  # untrusted-review context; these global CLI packages do not require them.
  && npm install --global --omit=dev --ignore-scripts \
    @anthropic-ai/claude-code@1.0.33 \
    @openai/codex@0.1.2505302106

RUN useradd --create-home --shell /bin/bash reviewer

ENV HOME=/home/reviewer \
  CODEX_HOME=/home/reviewer/.codex \
  CLAUDE_HOME=/home/reviewer/.claude \
  PAPERCLIP_HOME=/home/reviewer/.paperclip-review \
  PNPM_HOME=/home/reviewer/.local/share/pnpm \
  PATH=/home/reviewer/.local/share/pnpm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

WORKDIR /work

COPY --chown=reviewer:reviewer docker/untrusted-review/bin/review-checkout-pr /usr/local/bin/review-checkout-pr

RUN chmod +x /usr/local/bin/review-checkout-pr \
  && mkdir -p /work \
  && chown -R reviewer:reviewer /work

USER reviewer

EXPOSE 3100 5173

CMD ["bash", "-l"]
