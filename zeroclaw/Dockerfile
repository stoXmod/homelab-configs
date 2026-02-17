FROM debian:bookworm-slim

# Install dependencies for SSL/Networking
RUN apt-get update && apt-get install -y ca-certificates openssl && rm -rf /var/lib/apt/lists/*

# Create app directory
WORKDIR /app

# Copy binary and config
COPY zeroclaw /app/zeroclaw
COPY config.toml /root/.zeroclaw/config.toml

# Set permissions
RUN chmod +x /app/zeroclaw

# Expose Gateway Port
EXPOSE 8080

# Run the Daemon
CMD ["/app/zeroclaw", "daemon"]
