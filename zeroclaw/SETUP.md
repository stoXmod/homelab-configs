# ZeroClaw — Homelab Setup Guide

> Lightweight AI assistant agent for Raspberry Pi 3B+ (~5MB RAM)

---

## Prerequisites

- Raspberry Pi 3B+ (or newer) running **64-bit OS**
- Docker and Docker Compose installed
- An API key from [OpenRouter](https://openrouter.ai/) (or similar provider)

---

## Step 1 — Clone the Repo (if not already done)

```bash
git clone <repository-url>
cd homelab-configs/zeroclaw
```

---

## Step 2 — Create the Data Directory

```bash
mkdir -p data
```

---

## Step 3 — Download the ZeroClaw Binary

Download the pre-compiled ARM64 binary. **Do not compile from source on the Pi 3B+** — it will run out of memory.

```bash
# For 64-bit OS (aarch64)
wget https://github.com/zeroclaw-labs/zeroclaw/releases/latest/download/zeroclaw-aarch64-unknown-linux-gnu -O zeroclaw

# Make it executable
chmod +x zeroclaw
```

> **Note:** If running a 32-bit OS (check with `uname -m`), download the `armv7` version instead.

---

## Step 4 — Configure

Edit `config.toml` and replace the placeholder API key:

```toml
api_key = "your-actual-api-key-here"
```

| Setting               | Default                      | Description                       |
| --------------------- | ---------------------------- | --------------------------------- |
| `api_key`             | *(required)*                 | OpenRouter or compatible API key  |
| `default_model`       | `anthropic/claude-3-haiku`   | LLM model (Haiku is Pi-friendly)  |
| `default_temperature` | `0.7`                        | Response creativity (0.0–1.0)     |
| `require_pairing`     | `true`                       | Require pairing code on connect   |
| `backend`             | `sqlite`                     | Built-in SQLite vector database   |
| `level`               | `supervised`                 | Autonomy level                    |

---

## Step 5 — Build & Start

```bash
docker compose up -d --build
```

---

## Step 6 — Verify & Pair

### Check Logs

```bash
docker compose logs -f zeroclaw
```

Look for a **6-digit pairing code** in the output.

### Pair a Client

```bash
curl -X POST http://<raspberry-pi-ip>:8085/pair \
  -H "X-Pairing-Code: <CODE_FROM_LOGS>"
```

---

## Step 7 — (Optional) Nginx Proxy Manager HTTPS

If you have **Nginx Proxy Manager** running on port 81:

1. Open `http://<raspberry-pi-ip>:81`
2. **Proxy Hosts** → **Add Proxy Host**
3. Fill in:
   - **Domain:** `zeroclaw.yourdomain.com`
   - **Scheme:** `http`
   - **Forward Host:** `<raspberry-pi-ip>`
   - **Forward Port:** `8085`
4. **SSL** → Request a Let's Encrypt certificate
5. **Save**

---

## Port Reference

| Port | Protocol | Description         |
| ---- | -------- | ------------------- |
| 8085 | TCP      | Gateway / Webhook API |

---

## File Structure

```
zeroclaw/
├── config.toml          # Agent configuration
├── docker-compose.yml   # Docker Compose service definition
├── Dockerfile           # Container build (wraps binary)
├── data/                # Persistent workspace & memory (gitignored)
├── SETUP.md             # This file
└── zeroclaw.md          # Original setup reference
```

---

## Managing the Service

```bash
# Stop
docker compose down

# Restart
docker compose restart

# Update (re-download binary, then rebuild)
docker compose up -d --build

# View logs
docker compose logs -f
```

---

## Troubleshooting

| Issue                    | Fix                                                      |
| ------------------------ | -------------------------------------------------------- |
| Binary won't execute     | Check arch: `uname -m` — use matching binary             |
| Container keeps crashing | Check logs: `docker compose logs zeroclaw`                |
| Can't reach port 8085    | Verify: `sudo netstat -tlnp \| grep 8085`                |
| Pairing code not showing | Wait ~30s after startup, then check logs again            |
