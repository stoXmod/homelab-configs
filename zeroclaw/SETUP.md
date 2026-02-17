# ZeroClaw – Homelab Setup Guide

> Lightweight AI assistant agent for your Raspberry Pi homelab

---

## Overview

| Item             | Detail                                     |
| ---------------- | ------------------------------------------ |
| **Service**      | ZeroClaw AI Gateway                        |
| **Image**        | `ghcr.io/theonlyhennygod/zeroclaw:latest`  |
| **Host Port**    | `8085` → container port `3000`             |
| **RAM Usage**    | < 5 MB (128 MB container limit)            |
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
├── Dockerfile           # Fallback: build from source (not used by default)
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

Alternatively, set `API_KEY` as an environment variable (create a `.env` file):

```bash
echo "API_KEY=sk-or-v1-your-actual-key-here" > .env
```

### 3. Start the Service

```bash
docker compose up -d
```

This will:

1. Pull the official ZeroClaw image from GHCR
2. Mount `config.toml` at `/zeroclaw-data/.zeroclaw/config.toml`
3. Mount `data/` at `/zeroclaw-data/workspace`
4. Expose the gateway on host port **8085** (container port 3000)

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
| `workspace_dir`              | Workspace directory inside container            | `/zeroclaw-data/workspace` |
| `config_path`                | Config file path inside container               | `/zeroclaw-data/.zeroclaw/config.toml` |
| `api_key`                    | API key for the LLM provider                    | *(required)*         |
| `default_provider`           | LLM provider (`openrouter` / `anthropic`)       | `openrouter`         |
| `default_model`              | Model to use                                    | `anthropic/claude-3-haiku` |
| `default_temperature`        | LLM temperature                                 | `0.7`                |
| `memory.backend`             | Storage backend                                 | `sqlite`             |
| `memory.auto_save`           | Auto-persist conversations                      | `true`               |
| `gateway.port`               | Gateway listening port                           | `3000`               |
| `gateway.host`               | Gateway bind address                             | `[::]`               |
| `gateway.require_pairing`    | Require pairing code for first connect           | `true`               |
| `gateway.allow_public_bind`  | Allow binding to `0.0.0.0` / `[::]`             | `true`               |
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

# Update (pull latest image)
docker compose pull && docker compose up -d

# Check resource usage
docker stats zeroclaw
```

---

## Troubleshooting

### Container won't start

```bash
# Check logs
docker compose up  # (without -d to see output)

# Check if port is in use
sudo ss -tlnp | grep 8085
```

### Can't find pairing code

```bash
docker compose logs zeroclaw | grep -i pairing
```

### Out of memory on Pi 3B+

The container is limited to 128 MB. If your Pi is tight on RAM:

```bash
# Check overall memory
free -h

# Check per-container usage
docker stats --no-stream
```

Consider stopping heavy services temporarily if needed.

### Image pull fails

If GHCR is unreachable, you can build from source:

```bash
# Clone the source repo
git clone https://github.com/theonlyhennygod/zeroclaw.git zeroclaw-src

# Build locally
docker build -t zeroclaw-local -f Dockerfile ./zeroclaw-src

# Update docker-compose.yml to use local build:
# Comment out:  image: ghcr.io/theonlyhennygod/zeroclaw:latest
# Uncomment:    build: .
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
