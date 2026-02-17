# ZeroClaw – Homelab Setup Guide

> Lightweight AI assistant agent for your Raspberry Pi homelab

---

## Overview

| Item             | Detail                                     |
| ---------------- | ------------------------------------------ |
| **Service**      | ZeroClaw AI Gateway                        |
| **Port**         | `8085`                                     |
| **RAM Usage**    | < 5 MB (50 MB container limit)             |
| **Backend DB**   | SQLite (built-in)                          |
| **Architecture** | ARM64 (Raspberry Pi 3B+ / 4 / 5)          |
| **Security**     | Pairing-based authentication               |

---

## Prerequisites

- Docker & Docker Compose installed on the Pi
- An API key from [OpenRouter](https://openrouter.ai/) or direct Claude API key
- Port `8085` available (check with `sudo ss -tlnp | grep 8085`)

---

## Files in This Directory

```
zeroclaw/
├── config.toml          # Agent configuration
├── docker-compose.yml   # Docker Compose service definition
├── Dockerfile           # Container build (ARM64 binary)
├── data/                # Persistent workspace & SQLite memory
│   └── .gitkeep
├── guide.md             # Original setup reference
└── SETUP.md             # ← You are here
```

---

## Step-by-Step Deployment

### 1. Clone the Repo (if not already)

```bash
git clone <your-repo-url>
cd homelab-configs/zeroclaw
```

### 2. Set Your API Key

Edit `config.toml` and replace the placeholder:

```toml
api_key = "sk-or-v1-your-actual-key-here"
```

You can get a key from:

- **OpenRouter**: <https://openrouter.ai/keys> (recommended – access to multiple models)
- **Anthropic**: <https://console.anthropic.com/>

### 3. Build & Start

```bash
docker compose up -d --build
```

This will:

1. Build the container from `Dockerfile` (downloads the ARM64 binary)
2. Mount `config.toml` and `data/` directory
3. Expose the gateway on port **8085**

### 4. Get the Pairing Code

Since `require_pairing = true` in the config, you need a one-time code to connect:

```bash
docker compose logs zeroclaw
```

Look for a line like:

```
[AUTH] Pairing code: XXXXXX
```

### 5. Verify It's Running

```bash
# Check container status
docker ps | grep zeroclaw

# Test the gateway endpoint
curl http://localhost:8085
```

Access from any device on your network at:

```
http://<your-pi-ip>:8085
```

---

## Configuration Reference

### `config.toml` Options

| Key                          | Description                                     | Default              |
| ---------------------------- | ----------------------------------------------- | -------------------- |
| `api_key`                    | API key for the LLM provider                    | *(required)*         |
| `default_provider`           | LLM provider (`openrouter` / `anthropic`)       | `openrouter`         |
| `default_model`              | Model to use                                    | `anthropic/claude-3-haiku` |
| `memory.backend`             | Storage backend                                 | `sqlite`             |
| `memory.auto_save`           | Auto-persist conversations                      | `true`               |
| `gateway.require_pairing`    | Require pairing code for first connect           | `true`               |
| `gateway.allow_public_bind`  | Allow binding to `0.0.0.0`                       | `false`              |
| `autonomy.level`             | Autonomy level (`supervised` / `autonomous`)    | `supervised`         |
| `autonomy.workspace_only`    | Restrict actions to workspace                    | `true`               |
| `autonomy.allowed_commands`  | Shell commands ZeroClaw can execute              | `docker,ls,cat,grep,git` |

---

## Managing the Service

```bash
# Stop
docker compose down

# Restart
docker compose restart

# View live logs
docker compose logs -f zeroclaw

# Update (rebuild with latest binary)
docker compose up -d --build

# Check resource usage
docker stats zeroclaw
```

---

## Troubleshooting

### Container won't start

```bash
# Check build logs
docker compose up --build  # (without -d to see output)

# Check if port is in use
sudo ss -tlnp | grep 8085
```

### Can't find pairing code

```bash
docker compose logs zeroclaw | grep -i pairing
```

### Out of memory on Pi 3B+

The container is limited to 50 MB. If your Pi is tight on RAM:

```bash
# Check overall memory
free -h

# Check per-container usage
docker stats --no-stream
```

Consider stopping heavy services temporarily if needed.

### Binary download fails during build

If GitHub is unreachable, manually download and place the binary:

```bash
# On a machine with internet access
wget https://github.com/zeroclaw-labs/zeroclaw/releases/latest/download/zeroclaw-arm64

# Copy to Pi
scp zeroclaw-arm64 pi@<pi-ip>:~/homelab-configs/zeroclaw/

# Update Dockerfile to use COPY instead of ADD
# Replace the ADD line with:
# COPY zeroclaw-arm64 /usr/local/bin/zeroclaw
```

---

## Hardware Notes (Raspberry Pi 3B+)

> [!IMPORTANT]
> The Pi 3B+ has only **1 GB RAM**. ZeroClaw uses < 5 MB, but monitor your other services.

- **Memory**: Run `docker stats` to check all container usage
- **Storage**: Keep the `data/` folder on the fastest available storage (SSD > SD card)
- **Networking**: Ensure port `8085` is not claimed by another service (see [port allocation](../README.md#port-allocation))
- **Ask ZeroClaw**: Once running, you can ask it *"Which container is using the most RAM?"* to help manage your homelab

---

## Integration with Your Homelab

ZeroClaw has access to the Docker socket, so it can monitor and help manage your other services:

- View container status and logs
- Restart services
- Check resource usage
- Search configuration files

The `supervised` autonomy level means it will **ask for confirmation** before executing any commands.
