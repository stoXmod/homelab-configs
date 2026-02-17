# syntax=docker/dockerfile:1.7

# ZeroClaw Local Build Dockerfile
# Primarily use the pre-built image via docker-compose:
#   image: ghcr.io/theonlyhennygod/zeroclaw:latest
#
# This Dockerfile is a FALLBACK for building from source.
# Clone the source repo first:
#   git clone https://github.com/theonlyhennygod/zeroclaw.git zeroclaw-src
#   docker build -t zeroclaw-local -f Dockerfile ./zeroclaw-src

# ── Stage 1: Build ────────────────────────────────────────────
FROM rust:1.92-slim AS builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# 1. Copy manifests to cache dependencies
COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build --release --locked
RUN rm -rf src

# 2. Copy source code and build
COPY . .
RUN cargo build --release --locked && \
    cp target/release/zeroclaw /app/zeroclaw && \
    strip /app/zeroclaw

# ── Stage 2: Permissions & Config Prep ───────────────────────
FROM busybox:1.37 AS permissions
RUN mkdir -p /zeroclaw-data/.zeroclaw /zeroclaw-data/workspace

RUN cat > /zeroclaw-data/.zeroclaw/config.toml <<EOF
workspace_dir = "/zeroclaw-data/workspace"
config_path = "/zeroclaw-data/.zeroclaw/config.toml"
api_key = "${API_KEY:-REPLACE_ME}"
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-20250514"
default_temperature = 0.7

[gateway]
port = 3000
host = "[::]"
allow_public_bind = true
EOF

RUN chown -R 65534:65534 /zeroclaw-data

# ── Stage 3: Runtime (Distroless) ─────────────────────────────
FROM gcr.io/distroless/cc-debian13:nonroot AS release

COPY --from=builder /app/zeroclaw /usr/local/bin/zeroclaw
COPY --from=permissions /zeroclaw-data /zeroclaw-data

ENV ZEROCLAW_WORKSPACE=/zeroclaw-data/workspace
ENV HOME=/zeroclaw-data
ENV PROVIDER="openrouter"
ENV ZEROCLAW_GATEWAY_PORT=3000

WORKDIR /zeroclaw-data
USER 65534:65534
EXPOSE 3000
ENTRYPOINT ["zeroclaw"]
CMD ["gateway", "--port", "3000", "--host", "[::]"]