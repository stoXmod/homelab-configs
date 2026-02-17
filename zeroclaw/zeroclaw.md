This is a perfect fit for your Raspberry Pi 3B+ (1GB RAM). **ZeroClaw** is extremely lightweight (<5MB RAM usage), making it one of the few AI assistants that will run comfortably alongside your existing services without crashing the system.

Since your setup follows a strict Docker Compose structure, we will containerize the ZeroClaw binary. **Do not try to compile it from source on the Pi 3B+**, as the compilation process will likely run out of memory. We will use the pre-compiled binary method wrapped in Docker.

### 1. Port Allocation Update

First, we need to assign a unique port for ZeroClaw to avoid conflicts with your existing services (like CasaOS and qBittorrent which use 8080 internally).

| Port | Service | Description |
| --- | --- | --- |
| **8085** | **ZeroClaw** | **Gateway / Webhook API** |

---

### 2. Create Service Directory

Create a directory for ZeroClaw to match your existing homelab structure.

```bash
mkdir zeroclaw
cd zeroclaw
mkdir data

```

---

### 3. Download the ZeroClaw Binary

We will download the ARM64 binary (compatible with Raspberry Pi 3B+ running a 64-bit OS) directly.

> **Note:** If you are running a 32-bit OS (check with `uname -m`), download the `armv7` version instead.

```bash
# Download the latest release (replace URL with actual latest release link from ZeroClaw repo)
wget https://github.com/zeroclaw-labs/zeroclaw/releases/latest/download/zeroclaw-aarch64-unknown-linux-gnu -O zeroclaw

# Make it executable
chmod +x zeroclaw

```

---

### 4. Create the Configuration

Create a `config.toml` file in the `zeroclaw` directory. This configures the agent, memory, and security.

```bash
nano config.toml

```

**Paste the following:**

```toml
# ~/.zeroclaw/config.toml
api_key = "your-api-key-here" # Get from OpenRouter or similar
default_provider = "openrouter"
default_model = "anthropic/claude-3-haiku" # Lightweight model recommended for Pi
default_temperature = 0.7

[gateway]
require_pairing = true          # Security: Require pairing code on first connect
allow_public_bind = false       # Binds to 0.0.0.0 only inside container

[memory]
backend = "sqlite"              # Uses built-in SQLite vector db
auto_save = true

[autonomy]
level = "supervised"
workspace_only = true           # Sandbox agent to workspace

```

---

### 5. Create Docker Configuration

Since we are using a binary, we need a custom `Dockerfile` to wrap it.

**Create `Dockerfile`:**

```dockerfile
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

```

**Create `docker-compose.yml`:**

```yaml
version: "3.8"

services:
  zeroclaw:
    build: .
    container_name: zeroclaw
    restart: unless-stopped
    
    # Map Host Port 8085 to Container Port 8080 (Gateway)
    ports:
      - "8085:8080"
    
    # Persistent Data
    volumes:
      - ./data:/root/.zeroclaw/workspace # Agent workspace & memory
      - ./config.toml:/root/.zeroclaw/config.toml # Config file
    
    # Environment (Optional overrides)
    environment:
      - RUST_LOG=info

```

---

### 6. Start the Service

```bash
# Build and start the container
docker compose up -d --build

```

### 7. Verification & Pairing

Once running, ZeroClaw requires a pairing code for security.

1. **Check Logs for Pairing Code:**
```bash
docker compose logs -f zeroclaw

```


*Look for a 6-digit code in the logs.*
2. **Pairing:**
You can pair with the gateway using the API:
```bash
curl -X POST http://<raspberry-pi-ip>:8085/pair \
  -H "X-Pairing-Code: <CODE_FROM_LOGS>"

```



### 8. Integration with Nginx Proxy Manager (Recommended)

Since you already have NPM running on port 81, you should proxy ZeroClaw to secure it with HTTPS.

1. Log in to **Nginx Proxy Manager** (`http://<raspberry-pi-ip>:81`).
2. Click **Proxy Hosts** -> **Add Proxy Host**.
3. **Details:**
* **Domain Names:** `zeroclaw.yourdomain.com` (or IP)
* **Scheme:** `http`
* **Forward Host:** `<raspberry-pi-ip>` (or container IP if on same network)
* **Forward Port:** `8085`


4. **SSL:** Request a new Let's Encrypt certificate.
5. **Save**.

You now have a high-performance AI agent running on your Pi 3B+ using less than 5MB of RAM!