# Docker in Proxmox Guide

## Table of Contents

1. [Docker Installation](#docker-installation)
2. [Docker Basics](#docker-basics)
3. [Docker Compose](#docker-compose)
4. [Popular Docker Containers](#popular-docker-containers)
5. [Docker Networking](#docker-networking)
6. [Docker Storage](#docker-storage)
7. [Docker Security](#docker-security)
8. [Docker Management Tools](#docker-management-tools)
9. [Backup Docker](#backup-docker)
10. [Troubleshooting](#troubleshooting)

---

## Docker Installation

### Install Docker in Container

```bash
# Create LXC
pct create 200 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:20 \
  --hostname docker-host \
  --memory 4096 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1,keyctl=1,fuse=1

# Start container
pct start 200

# Install Docker
pct exec 200 -- bash -c "
apt update
apt install -y ca-certificates curl gnupg lsb-release

# Add Docker repo
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable' > /etc/apt/sources.list.d/docker.list

# Install Docker
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Enable Docker
systemctl enable docker
systemctl start docker
"

# Add user to docker group
pct exec 200 -- usermod -aG docker root
```

### Using community-scripts

```bash
# One-command Docker install
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)"
```

---

## Docker Basics

### Basic Commands

```bash
# Docker version
docker --version

# Run container
docker run -d --name nginx nginx:latest

# List containers
docker ps -a

# List images
docker images

# Start/Stop/Remove
docker start nginx
docker stop nginx
docker rm nginx

# Execute in container
docker exec -it nginx bash

# View logs
docker logs -f nginx

# Pull image
docker pull nginx:latest
```

### Docker Container Lifecycle

```bash
# Run with restart
docker run -d --name app --restart unless-stopped app:latest

# Run with port mapping
docker run -d --name web -p 8080:80 nginx

# Run with volume
docker run -d --name data -v /data:/data postgres

# Run with environment
docker run -d --name db -e POSTGRES_PASSWORD=secret postgres

# Run with network
docker run -d --name app --network mynetwork nginx
```

---

## Docker Compose

### Installation

```bash
# Install Docker Compose
apt install -y docker-compose

# Or latest
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### Compose File

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    image: nginx:latest
    container_name: myapp
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - ./data:/data
    environment:
      - APP_ENV=production
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge
```

### Compose Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Rebuild
docker-compose up -d --build

# Pull latest
docker-compose pull
```

---

## Popular Docker Containers

### Web Servers & Proxies

```bash
# Nginx
docker run -d --name nginx -p 80:80 -p 443:443 nginx

# Nginx Proxy Manager
docker run -d --name npm \
  -p 80:80 -p 443:443 \
  -p 81:81 \
  -v npm-data:/data \
  --restart unless-stopped \
  jc21/nginx-proxy-manager

# Traefik
docker run -d --name traefik \
  -p 80:80 -p 443:443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v traefik-data:/etc/traefik \
  --restart unless-stopped \
  traefik:v2.9
```

### Databases

```bash
# PostgreSQL
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres

# MariaDB
docker run -d --name mariadb \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=mydb \
  -v mariadb-data:/var/lib/mysql \
  -p 3306:3306 \
  mariadb

# Redis
docker run -d --name redis \
  -v redis-data:/data \
  -p 6379:6379 \
  redis:latest --appendonly yes
```

### Monitoring

```bash
# Portainer
docker run -d --name portainer \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer-data:/data \
  -p 9000:9000 \
  --restart unless-stopped \
  portainer/portainer-ce

# Uptime Kuma
docker run -d --name kuma \
  -v kuma-data:/app/data \
  -p 3001:3001 \
  --restart unless-stopped \
  louislam/uptime-kuma

# Netdata
docker run -d --name netdata \
  -v netdata-proc:/host/proc:ro \
  -v netdata-sys:/host/sys:ro \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -p 19999:19999 \
  --restart unless-stopped \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  netdata/netdata
```

### Home Automation

```bash
# Home Assistant
docker run -d --name homeassistant \
  -v /home/homeassistant/config:/config \
  --network host \
  --restart unless-stopped \
  homeassistant/home-assistant:latest

# Zigbee2MQTT
docker run -d --name zigbee2mqtt \
  -v /opt/zigbee2mqtt/data:/data \
  -p 8080:8080 \
  --device /dev/ttyUSB0:/dev/ttyUSB0 \
  --restart unless-stopped \
  koenkk/zigbee2mqtt

# Node-RED
docker run -d --name nodered \
  -v nodered-data:/data \
  -p 1880:1880 \
  --restart unless-stopped \
  nodered/node-red
```

### Media

```bash
# Jellyfin
docker run -d --name jellyfin \
  -v jellyfin-config:/config \
  -v jellyfin-cache:/cache \
  -v /mnt/media:/media \
  -p 8096:8096 -p 8920:8920 \
  --restart unless-stopped \
  jellyfin/jellyfin:latest

# Radarr
docker run -d --name radarr \
  -v radarr-config:/config \
  -v /mnt/media:/media \
  -p 7878:7878 \
  --restart unless-stopped \
  linuxserver/radarr

# qBittorrent
docker run -d --name qbittorrent \
  -v qbittorrent-config:/config \
  -v /mnt/downloads:/downloads \
  -p 6881:6881 -p 6881:6881/udp \
  -p 8080:8080 \
  --restart unless-stopped \
  linuxserver/qbittorrent
```

### Security

```bash
# Vaultwarden
docker run -d --name vaultwarden \
  -v vw-data:/data \
  -p 80:80 \
  --restart unless-stopped \
  vaultwarden/server:latest

# Authelia
docker run -d --name authelia \
  -v authelia-config:/config \
  -p 9091:9091 \
  --restart unless-stopped \
  authelia/authelia:latest

# CrowdSec
docker run -d --name crowdsec \
  -v crowdsec-config:/etc/crowdsec \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 8080:8080 \
  --restart unless-stopped \
  crowdsecurity/crowdsec
```

### Development

```bash
# Code Server
docker run -d --name codeserver \
  -v code-server-data:/home/coder \
  -p 8443:8443 \
  --restart unless-stopped \
  codercom/code-server:latest

# Gitea
docker run -d --name gitea \
  -v gitea-data:/data \
  -p 3000:3000 -p 22:22 \
  --restart unless-stopped \
  gitea/gitea:latest

# n8n
docker run -d --name n8n \
  -v n8n-data:/home/node/.n8n \
  -p 5678:5678 \
  --restart unless-stopped \
  n8nio/n8n
```

---

## Docker Networking

### Create Network

```bash
# Create bridge network
docker network create mynetwork

# Create overlay network (cluster)
docker network create -d overlay myoverlay

# Create macvlan
docker network create -d macvlan \
  --subnet 192.168.1.0/24 \
  --gateway 192.168.1.1 \
  -o parent=eth0 mymacvlan
```

### Connect Containers

```bash
# Connect to network
docker network connect mynetwork container

# Disconnect
docker network disconnect mynetwork container
```

### Port Mapping

```bash
# Random port
-P

# Specific port
-p 8080:80

# Random UDP
-p 53:53/udp

# Specific IP
-p 192.168.1.100:8080:80
```

---

## Docker Storage

### Volume Types

```bash
# Named volume
-v myvolume:/data

# Bind mount
-v /host/path:/container/path

# tmpfs (memory)
--tmpfs /data
```

### Backup Volumes

```bash
# Backup volume
docker run --rm -v backup:/data -v /backup:/backup alpine tar -czf /backup/backup.tar.gz -C /data .

# Restore
docker run --rm -v backup:/data -v /backup:/backup alpine tar -xzf /backup/backup.tar.gz -C /data
```

---

## Docker Security

### Security Best Practices

```bash
# Run as non-root
--user 1000:1000

# Read only
--read-only

# No new privileges
--security-opt no-new-privileges:true

# Limit capabilities
--cap-drop ALL
--cap-add NET_BIND_SERVICE

# SELinux/AppArmor
--security-opt apparmor=unconfined
```

### Scan for Vulnerabilities

```bash
# Install Trivy
docker run --rm aquasec/trivy image nginx:latest

# Check for CVEs
trivy image --severity HIGH,CRITICAL nginx:latest
```

---

## Docker Management Tools

### Portainer

```bash
# Install
docker run -d --name portainer \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer-data:/data \
  -p 9000:9000 \
  --restart unless-stopped \
  portainer/portainer-ce:latest
```

### Yacht

```bash
# Install
docker run -d --name yacht \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v yacht-data:/config \
  -p 8000:8000 \
  --restart unless-stopped \
  Selfhosted/Yacht
```

### Lazydocker

```bash
# Install
docker run --detach \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/.ladocker-data:/root/.docker \
  lazyteam/lazydocker

# Use
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock lazyteam/lazydocker
```

---

## Backup Docker

### Backup Containers

```bash
# Backup all volumes
docker run --rm \
  -v $(pwd)/backups:/backups \
  -v app-data:/data alpine \
  tar -czf /backups/app-data-$(date +%Y%m%d).tar.gz -C /data .

# Backup configs
cp docker-compose.yml backups/
tar -czf backups/docker-compose-$(date +%Y%m%d).tar.gz backups/
```

### Restore

```bash
# Restore volumes
docker volume create app-data
docker run --rm \
  -v app-data:/data \
  -v $(pwd)/backups:/backup \
  alpine tar -xzf /backup/app-data-20240101.tar.gz -C /data

# Restore compose
tar -xzf backups/docker-compose-20240101.tar.gz -C backups/
docker-compose up -d
```

---

## Troubleshooting

### View Logs

```bash
# Container logs
docker logs container
docker logs -f container

# All containers
docker-compose logs -f

# System logs
journalctl -u docker
```

### Resource Usage

```bash
# Docker stats
docker stats

# Top
docker top container

# System df
docker system df
```

### Cleanup

```bash
# Remove stopped containers
docker container prune -f

# Remove unused images
docker image prune -f

# Remove unused volumes
docker volume prune -f

# Full cleanup
docker system prune -a
```

### Network Issues

```bash
# Inspect network
docker network inspect mynetwork

# Debug DNS
docker exec container nslookup hostname

# Check iptables
iptables -L -n -v
```

### Common Errors

```bash
# Port already in use
# Solution: Use different port or stop existing service

# Volume permission denied
# Solution: chmod volume or run container as root

# Network not found
# Solution: Create network or connect
```

---

## Quick Reference

| Task | Command |
|------|---------|
| List containers | `docker ps` |
| List images | `docker images` |
| Start container | `docker start name` |
| Stop container | `docker stop name` |
| View logs | `docker logs -f name` |
| Exec into | `docker exec -it name bash` |
| Compose up | `docker-compose up -d` |
| Compose down | `docker-compose down` |
| Stats | `docker stats` |
| Prune | `docker system prune -a` |

## Keywords

#docker #containers #docker-compose #portainer #selfhosted #homelab

## Links

- [[Proxmox-VE]] - Main documentation
- [[Containers]] - LXC containers
- [[HomeLab]] - Home lab setup
- [[Community-Scripts]] - One-click installs