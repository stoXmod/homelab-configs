# Homelab Docker Configurations

> A collection of Docker containers for self-hosted services on Raspberry Pi systems

This repository contains Docker Compose configurations for running various self-hosted services on Raspberry Pi devices. Each service is organized in its own directory with persistent data volumes and consistent configuration patterns.

## Table of Contents

- [Overview](#overview)
- [Container Services](#container-services)
  - [CasaOS](#casaos)
  - [Jellyfin](#jellyfin)
  - [Pi-hole](#pi-hole)
  - [Nginx Proxy Manager (NPM)](#nginx-proxy-manager-npm)
  - [n8n](#n8n)
  - [LGTM Stack](#lgtm-stack)
- [Getting Started](#getting-started)
- [Managing Containers](#managing-containers)
- [Adding New Containers](#adding-new-containers)
- [Raspberry Pi Considerations](#raspberry-pi-considerations)
- [Troubleshooting](#troubleshooting)

---

## Overview

This setup provides a complete homelab environment with:
- **Media Server**: Jellyfin for streaming
- **Network Management**: Pi-hole for DNS-based ad blocking
- **Reverse Proxy**: Nginx Proxy Manager for HTTPS and domain routing
- **Automation**: n8n for workflow automation
- **Monitoring**: LGTM stack (Loki, Grafana, Tempo, Mimir) for observability
- **Container Management**: CasaOS for easy Docker management

All services are configured to run on Raspberry Pi with appropriate resource considerations and hardware acceleration where applicable.

---

## Container Services

### CasaOS

**Purpose**: Web-based Docker container management interface

**Location**: `/casa` and `/casaos`

**Docker Image**: `dockurr/casa`

**Access**: `http://<raspberry-pi-ip>:9090`

**Configuration**:
```yaml
ports:
  - 9090:8080
volumes:
  - ./casa:/DATA
  - /var/run/docker.sock:/var/run/docker.sock  # Docker socket access
```

**Description**: CasaOS provides a user-friendly web interface for managing Docker containers. It has access to the Docker socket, allowing it to start, stop, and monitor other containers on your system.

---

### Jellyfin

**Purpose**: Media server for streaming movies, TV shows, and music

**Location**: `/jellyfin`

**Docker Image**: `jellyfin/jellyfin:latest`

**Access**: `http://<raspberry-pi-ip>:8096`

**Configuration**:
```yaml
network_mode: host
user: 1000:1000  # Replace with your user ID
volumes:
  - ./config:/config
  - ./cache:/cache
  - ~/media:/media:ro
  - /home/stoxmod/media/movies:/Movies:ro
  - /mnt/passport:/Passport:ro
devices:
  - /dev/dri:/dev/dri  # Hardware acceleration for Pi 5
```

**Features**:
- Hardware-accelerated video transcoding (Raspberry Pi 5 VideoCore VII)
- Network mode: `host` for optimal performance
- Read-only media mounts for data protection
- Customizable media library paths

**Setup Notes**:
1. Update `user` with your user ID (run `id -u` and `id -g`)
2. Update `JELLYFIN_PublishedServerUrl` with your Pi's IP address
3. Customize media volume paths to match your setup

---

### Pi-hole

**Purpose**: Network-wide ad blocking and DNS server

**Location**: `/pihole`

**Docker Image**: `pihole/pihole:latest`

**Access**: `http://<raspberry-pi-ip>:8443/admin`

**Configuration**:
```yaml
network_mode: host
environment:
  TZ: 'Asia/Colombo'
  WEBPASSWORD: '<your-password>'
  FTLCONF_LOCAL_IPV4: '192.168.1.174'
  PIHOLE_DNS_: '1.1.1.1;1.0.0.1'  # Cloudflare DNS
  WEB_PORT: 8443
volumes:
  - ./etc-pihole:/etc/pihole
  - ./etc-dnsmasq.d:/etc/dnsmasq.d
```

**Features**:
- Network-wide ad blocking
- Custom DNS server
- Web interface on port 8443
- Persistent configuration and block lists

**Setup Notes**:
1. Change `WEBPASSWORD` to your desired admin password
2. Update `FTLCONF_LOCAL_IPV4` with your Pi's static IP address
3. Configure your router to use Pi-hole as the DNS server

---

### Nginx Proxy Manager (NPM)

**Purpose**: Reverse proxy with SSL certificate management

**Location**: `/npm`

**Docker Image**: `jc21/nginx-proxy-manager:latest`

**Access**: 
- Admin UI: `http://<raspberry-pi-ip>:81`
- HTTP: Port 80
- HTTPS: Port 443

**Configuration**:
```yaml
ports:
  - '80:80'    # Public HTTP
  - '443:443'  # Public HTTPS
  - '81:81'    # Admin interface
volumes:
  - ./data:/data
  - ./letsencrypt:/etc/letsencrypt
```

**Features**:
- Easy SSL certificate management with Let's Encrypt
- Reverse proxy for all your services
- Web-based configuration interface
- Access lists for security

**Default Credentials**:
- Email: `admin@example.com`
- Password: `changeme`
- **Change these immediately after first login!**

---

### n8n

**Purpose**: Workflow automation platform

**Location**: `/n8n`

**Docker Image**: `docker.n8n.io/n8nio/n8n`

**Access**: `http://<raspberry-pi-ip>:5678`

**Configuration**:
```yaml
ports:
  - "5678:5678"
environment:
  - GENERIC_TIMEZONE=Asia/Colombo
  - TZ=Asia/Colombo
volumes:
  - n8n_data:/home/node/.n8n
```

**Features**:
- Visual workflow builder
- Hundreds of integrations
- Persistent workflow storage
- Timezone configuration

**Optional Settings**:
Uncomment in the docker-compose.yml to enable:
- Basic authentication
- Custom webhook URL for domain access

---

### LGTM Stack

**Purpose**: Complete observability stack (Loki, Grafana, Tempo, Mimir, Prometheus)

**Location**: `/lgtm`

**Docker Images**: 
- `grafana/otel-lgtm` (main stack)
- `prom/node-exporter` (system metrics)

**Access**:
- Grafana: `http://<raspberry-pi-ip>:3000`
- Prometheus: `http://<raspberry-pi-ip>:9092`

**Configuration**:
```yaml
ports:
  - "3000:3000"   # Grafana UI
  - "4317:4317"   # OTLP gRPC
  - "4318:4318"   # OTLP HTTP
  - "9092:9090"   # Prometheus
  - "3100:3100"   # Loki
volumes:
  - otel-lgtm-data:/data
  - ./lgtm-config.yaml:/otel-lgtm/prometheus.yaml:ro
  - /sys:/host/sys:ro
  - /proc:/host/proc:ro
```

**Features**:
- **Grafana**: Visualization and dashboards
- **Prometheus**: Metrics collection
- **Loki**: Log aggregation
- **Tempo**: Distributed tracing
- **Node Exporter**: System metrics (CPU, memory, disk, temperature)
- Anonymous viewer access enabled

**Monitored Metrics**:
- System resources (CPU, memory, disk)
- Hardware temperatures and thermal zones
- Container performance
- Custom application metrics

---

## Getting Started

### Prerequisites

1. **Raspberry Pi** (tested on Pi 5)
2. **Docker** and **Docker Compose** installed
3. **Static IP address** configured for your Pi
4. **Sufficient storage** for media and data

### Initial Setup

1. **Clone this repository**:
   ```bash
   git clone <repository-url>
   cd homelab-configs
   ```

2. **Configure environment-specific settings**:
   - Update IP addresses in configurations
   - Set passwords and credentials
   - Adjust volume paths for your system

3. **Start all services**:
   ```bash
   # Start all services
   for dir in */; do
     (cd "$dir" && docker compose up -d)
   done
   ```

   Or start individual services:
   ```bash
   cd jellyfin
   docker compose up -d
   ```

---

## Managing Containers

### Start a Service

```bash
cd <service-directory>
docker compose up -d
```

### Stop a Service

```bash
cd <service-directory>
docker compose down
```

### View Logs

```bash
cd <service-directory>
docker compose logs -f
```

### Restart a Service

```bash
cd <service-directory>
docker compose restart
```

### Update a Service

```bash
cd <service-directory>
docker compose pull
docker compose up -d
```

### Check Service Status

```bash
docker ps
```

---

## Adding New Containers

Follow this standardized structure when adding new services:

### 1. Create Service Directory

```bash
mkdir new-service
cd new-service
```

### 2. Create docker-compose.yml

Use this template:

```yaml
version: '3.8'

services:
  service-name:
    image: <docker-image>:latest
    container_name: service-name
    restart: unless-stopped
    
    # Ports (if needed)
    ports:
      - "host-port:container-port"
    
    # Environment variables
    environment:
      - TZ=Asia/Colombo
      # Add service-specific variables
    
    # Volumes for persistent data
    volumes:
      - ./data:/data
      # Add service-specific volumes
    
    # Network (optional)
    # networks:
    #   - service-network

# Define volumes (if using named volumes)
volumes:
  service-data:
    driver: local

# Define networks (if needed)
# networks:
#   service-network:
#     driver: bridge
```

### 3. Directory Structure

Organize your service following this pattern:

```
new-service/
├── docker-compose.yml
├── data/              # Application data
├── config/            # Configuration files
└── logs/              # Log files (if needed)
```

### 4. Configuration Best Practices

- **Restart Policy**: Use `unless-stopped` for most services
- **Timezone**: Set `TZ` environment variable
- **Volumes**: Use relative paths (`./`) for easy portability
- **Ports**: Document all exposed ports in comments
- **Security**: Never commit passwords; use environment files
- **User Permissions**: Set appropriate user IDs for file access

### 5. Test and Verify

```bash
# Validate configuration
docker compose config

# Start in foreground to check for errors
docker compose up

# If successful, run in background
docker compose up -d

# Check logs
docker compose logs -f
```

### 6. Document Your Service

Add a section to this README documenting:
- Service purpose
- Access URLs and ports
- Important configuration options
- Any special setup requirements

---

## Raspberry Pi Considerations

### Hardware Requirements

- **Minimum**: Raspberry Pi 4 with 4GB RAM
- **Recommended**: Raspberry Pi 5 with 8GB RAM
- **Storage**: SSD recommended for better performance (vs SD card)
- **Cooling**: Heatsink or fan recommended under load

### Performance Optimization

1. **Use Hardware Acceleration**:
   - Jellyfin uses `/dev/dri` for video transcoding
   - Check available devices: `ls -la /dev/dri`

2. **Network Mode**:
   - Some services use `host` network mode for better performance
   - Trade-off: Port conflicts are possible

3. **Resource Limits**:
   Add to docker-compose.yml if needed:
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '2'
         memory: 1G
   ```

4. **Storage Optimization**:
   - Mount external drives for media storage
   - Use SSD for Docker volumes
   - Regular cleanup: `docker system prune`

### ARM Architecture

All images in this setup support ARM architecture (`arm64`). When adding new services, verify ARM support:

```bash
docker manifest inspect <image-name>:latest | grep architecture
```

### Power Management

Services are configured with `restart: unless-stopped` to survive reboots. To disable auto-start:

```yaml
restart: "no"
```

---

## Troubleshooting

### Service Won't Start

1. Check logs:
   ```bash
   docker compose logs
   ```

2. Verify port availability:
   ```bash
   sudo netstat -tlnp | grep <port>
   ```

3. Check file permissions:
   ```bash
   ls -la
   sudo chown -R 1000:1000 ./data
   ```

### Performance Issues

1. Monitor resources:
   ```bash
   docker stats
   ```

2. Check Pi temperature:
   ```bash
   vcgencmd measure_temp
   ```

3. Monitor disk I/O:
   ```bash
   iostat -x 1
   ```

### Network Issues

1. Check container networking:
   ```bash
   docker network ls
   docker network inspect <network-name>
   ```

2. Verify DNS resolution:
   ```bash
   docker exec <container> ping google.com
   ```

### Permission Errors

1. Find your user/group ID:
   ```bash
   id -u  # User ID
   id -g  # Group ID
   ```

2. Update ownership:
   ```bash
   sudo chown -R $(id -u):$(id -g) ./directory
   ```

### Container Logs

View detailed logs with timestamps:
```bash
docker compose logs -f --timestamps --tail=100
```

---

## Backup and Maintenance

### Backup Configuration

```bash
# Backup all configurations and data
tar -czf homelab-backup-$(date +%Y%m%d).tar.gz \
  --exclude='*/cache/*' \
  --exclude='*/logs/*' \
  homelab-configs/
```

### Update All Services

```bash
# Update all services
for dir in */; do
  echo "Updating $dir..."
  (cd "$dir" && docker compose pull && docker compose up -d)
done
```

### Cleanup

```bash
# Remove unused images and volumes
docker system prune -a --volumes
```

---

## Security Notes

- Change default passwords immediately
- Use Nginx Proxy Manager to add HTTPS
- Consider setting up Tailscale or WireGuard for remote access
- Enable UFW firewall for additional security
- Keep Docker and images updated regularly

---

## Contributing

When adding new services:
1. Follow the directory structure
2. Document in this README
3. Test thoroughly on Raspberry Pi
4. Include resource requirements

---

## License

Personal homelab configuration - use at your own discretion.

---

## Support

For issues with specific services, consult their official documentation:
- [CasaOS](https://casaos.io/)
- [Jellyfin](https://jellyfin.org/docs/)
- [Pi-hole](https://docs.pi-hole.net/)
- [Nginx Proxy Manager](https://nginxproxymanager.com/)
- [n8n](https://docs.n8n.io/)
- [Grafana](https://grafana.com/docs/)
