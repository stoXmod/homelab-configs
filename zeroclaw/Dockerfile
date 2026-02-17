FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates docker.io && rm -rf /var/lib/apt/lists/*
# Download the ARM64 binary directly (update version as needed)
ADD https://github.com/zeroclaw-labs/zeroclaw/releases/latest/download/zeroclaw-arm64 /usr/local/bin/zeroclaw
RUN chmod +x /usr/local/bin/zeroclaw
ENTRYPOINT ["zeroclaw", "gateway"]
