Setting up **ZeroClaw** on a Raspberry Pi 3B+ is a smart move. Since you only have **1GB of RAM** and are already running heavy services like Appwrite, OpenProject, and the LGTM stack, ZeroClaw’s ultra-lightweight footprint (<5MB RAM) is perfect for your environment.

Based on your current directory structure, here is the step-by-step guide to integrating ZeroClaw into your homelab.

---

## 1. Create the Service Directory

First, align ZeroClaw with your existing "one folder per service" pattern.

```bash
mkdir zeroclaw
cd zeroclaw
mkdir data

```

## 2. Configuration (`config.toml`)

Create a `config.toml` file inside the `zeroclaw` folder. Given your Pi's limited resources, we will use the **SQLite** backend which is built for efficiency.

```toml
api_key = "your-openrouter-or-claude-key"
default_provider = "openrouter"
default_model = "anthropic/claude-3-haiku" # Recommended for speed/low cost

[memory]
backend = "sqlite"
auto_save = true

[gateway]
require_pairing = true
allow_public_bind = false

[autonomy]
level = "supervised"
workspace_only = true
# Allow ZeroClaw to help manage your existing containers
allowed_commands = ["docker", "ls", "cat", "grep", "git"]

```

## 3. Docker Compose Setup

Create a `docker-compose.yml` file in the `zeroclaw` directory. This uses a custom build to ensure it's optimized for your Pi's ARM64 architecture.

```yaml
version: "3.8"

services:
  zeroclaw:
    build: .
    container_name: zeroclaw
    restart: unless-stopped
    ports:
      - "8085:8080" # Matches your README port allocation
    volumes:
      - ./data:/root/.zeroclaw/workspace
      - ./config.toml:/root/.zeroclaw/config.toml
      # Essential: give ZeroClaw access to manage your homelab
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - RUST_LOG=info
    deploy:
      resources:
        limits:
          memory: 50M # Even 50MB is overkill for ZeroClaw

```

## 4. Prepare the Dockerfile

Since you are running on a Pi 3B+, it is best to use the pre-compiled binary approach mentioned in your documentation to avoid heavy compilation on the Pi itself.

**Create a `Dockerfile`:**

```dockerfile
FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates docker.io && rm -rf /var/lib/apt/lists/*
# Download the ARM64 binary directly (update version as needed)
ADD https://github.com/zeroclaw-labs/zeroclaw/releases/latest/download/zeroclaw-arm64 /usr/local/bin/zeroclaw
RUN chmod +x /usr/local/bin/zeroclaw
ENTRYPOINT ["zeroclaw", "gateway"]

```

---

## 5. Deployment & Pairing

Start the service using your existing workflow:

```bash
docker compose up -d --build

```

### Security Pairing

Because you have `require_pairing = true`, you need the one-time code from the logs to connect your UI or API:

1. **Get the code:** `docker compose logs zeroclaw`
2. **Look for:** `[AUTH] Pairing code: XXXXXX`
3. **Access:** The gateway will be available at `http://<your-pi-ip>:8085`.

---

## Hardware Optimization for Pi 3B+

Since you are pushing that 1GB RAM to its limit with **Appwrite** and **OpenProject**:

* **Memory Management:** ZeroClaw is the "manager." Use it to monitor your other containers. You can ask it: *"Which of my containers is using the most RAM right now?"*
* **Storage:** Ensure your `zeroclaw/data` folder is on your SSD/SD card. The SQLite database for memory is very small but performs better on faster storage.
* **Port Conflict Check:** Your `README.md` already lists `8085` for ZeroClaw. Ensure no other service has snatched it during your manual tests.

**Would you like me to generate a specific `IDENTITY.md` for ZeroClaw so it knows it's the administrator of your Raspberry Pi homelab?**