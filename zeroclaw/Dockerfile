FROM rust:1.76-slim-bookworm as builder
RUN apt-get update && apt-get install -y pkg-config libssl-dev git && rm -rf /var/lib/apt/lists/*
WORKDIR /usr/src/zeroclaw
# Clone and build the lightweight Rust binary
RUN git clone https://github.com/zeroclaw-labs/zeroclaw.git . && \
    cargo build --release --locked

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates docker.io && rm -rf /var/lib/apt/lists/*
# Copy the tiny binary from the builder
COPY --from=builder /usr/src/zeroclaw/target/release/zeroclaw /usr/local/bin/zeroclaw
ENTRYPOINT ["zeroclaw", "gateway"]