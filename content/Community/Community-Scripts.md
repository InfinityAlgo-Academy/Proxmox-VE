# Proxmox VE Community Scripts - Complete Guide

**Official Website**: https://community-scripts.org/  
**GitHub**: https://github.com/community-scripts/ProxmoxVE  
**Total Scripts**: 558+ | **Installs**: 2.5M+

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Installation Methods](#installation-methods)
3. [Categories Overview](#categories-overview)
4. [Container & Docker](#container--docker)
5. [Home Automation](#home-automation)
6. [Media Servers](#media-servers)
7. [Networking](#networking)
8. [Monitoring & Analytics](#monitoring--analytics)
9. [Databases](#databases)
10. [Security](#security)
11. [Development Tools](#development-tools)
12. [Documents & Notes](#documents--notes)
13. [MQTT & Messaging](#mqtt--messaging)
14. [Automation & Scheduling](#automation--scheduling)
15. [NVR & Cameras](#nvr--cameras)
16. [Webservers & Proxies](#webservers--proxies)
17. [Proxmox & Virtualization](#proxmox--virtualization)
18. [Miscellaneous](#miscellaneous)
19. [Script Management](#script-management)
20. [Function Libraries](#function-libraries)

---

## Getting Started

### What Are Community Scripts?

Community Scripts are **one-command installation scripts** for Proxmox VE that automate the setup of popular self-hosted services. No manual configuration needed - just paste the command and follow prompts.

### How It Works

1. Visit **community-scripts.org**
2. Search for your service (e.g., "Home Assistant")
3. Copy the install command
4. Paste in Proxmox Shell
5. Follow interactive prompts

---

## Installation Methods

### Method 1: Web Installer

```bash
# 1. Go to community-scripts.org
# 2. Search for script
# 3. Copy command
# 4. Paste in Proxmox Shell
```

### Method 2: PVE Scripts Local

```bash
# Install local script manager
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/pve-scripts-local.sh)"
```

This adds a **menu to Proxmox UI** for easy script access.

### Method 3: Direct GitHub

```bash
# Clone repository
git clone https://github.com/community-scripts/ProxmoxVE.git /opt/pve-scripts

# Browse scripts
ls -la /opt/pve-scripts/ct/
```

---

## Categories Overview

| Category | Scripts | Description |
|----------|---------|-------------|
| Containers & Docker | 80+ | Docker-based services |
| Home Automation | 15+ | Smart home software |
| Media | 20+ | Media servers & downloaders |
| Networking | 25+ | DNS, VPN, proxies |
| Monitoring | 15+ | Metrics & uptime |
| Databases | 10+ | DB servers |
| Security | 15+ | Auth, security tools |
| Development | 20+ | Dev tools |
| Documents | 10+ | Document management |
| MQTT & Messaging | 10+ | IoT messaging |
| Automation | 10+ | Workflow tools |
| NVR & Cameras | 5+ | Camera systems |
| Webservers | 10+ | Web servers |
| Proxmox & VM | 10+ | PVE tools |
| Miscellaneous | 20+ | Other tools |

---

## Container & Docker

### Popular Docker Containers

| Script | Description | Ports |
|-------|-------------|------|
| Docker | Docker runtime | - |
| Portainer | Container management | 9000 |
| Docker Compose | Compose plugin | - |
| CasaOS | Simple OS interface | 3000 |
| Yacht | Docker manager | 8000 |
| Podman | Docker alternative | - |
| Rancher | K8s management | 80/443 |

### Installation

```bash
# Docker
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)"

# Portainer
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/portainer.sh)"

# CasaOS
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/casaos.sh)"
```

---

## Home Automation

### Smart Home Software

| Script | Description | Ports |
|-------|-------------|------|
| Home Assistant | Home automation | 8123 |
| Home Assistant Core | HA Python version | 5000 |
| Zigbee2MQTT | Zigbee bridge | 8080 |
| ESPHome | ESP device firmware | 6050 |
| Node-RED | Flow automation | 1880 |
| ioBroker | Automation | 8087 |
|openHAB | Home automation | 8080 |
| smartHome | Smart home | 3000 |

### Installation

```bash
# Home Assistant
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/homeassistant.sh)"

# Zigbee2MQTT
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/zigbee2mqtt.sh)"

# Node-RED
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/nodered.sh)"

# ESPHome
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/esphome.sh)"
```

---

## Media Servers

### Media Servers

| Script | Description | Ports |
|-------|-------------|------|
| Jellyfin | Media server | 8096 |
| Plex | Media server | 32400 |
| Emby | Media server | 8096 |
| MediaMob | Media list | 8090 |
| Harry Potter | Media downloader | - |

### Media Management

| Script | Description | Ports |
|-------|-------------|------|
| Radarr | Movie downloader | 7878 |
| Sonarr | TV downloader | 8989 |
| Lidarr | Music downloader | 8686 |
| Readarr | Books downloader | 8787 |
| Bazarr | Subtitle downloader | 6767 |
| Prowlarr | Indexer manager | 9696 |
| Jackett | Indexer proxy | 9117 |
| NZBGet | Usenet downloader | 6789 |
| SABnzbd | Usenet downloader | 8080 |

### Installation

```bash
# Jellyfin
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/jellyfin.sh)"

# Radarr
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/radarr.sh)"

# Sonarr
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/sonarr.sh)"

# Plex
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/plex.sh)"
```

---

## Networking

### DNS & Ad Block

| Script | Description | Ports |
|-------|-------------|------|
| Pi-hole | Ad blocker | 80/443 |
| AdGuard Home | Ad blocker | 3000 |
| AdGuard Home Beta | Beta version | 3000 |
| AdGuard Home CLI | CLI ad blocker | - |
| Blocky | DNS proxy | 1912 |

### VPN

| Script | Description | Ports |
|-------|-------------|------|
| WireGuard | VPN | 51820 |
| OpenVPN | VPN | 1194 |
| Tailscale | Zero-config VPN | - |
| Gluetun | VPN client | 5000 |
| Streisand | VPN automator | - |

### Network Tools

| Script | Description | Ports |
|-------|-------------|------|
| ddclient | Dynamic DNS | 19999 |
| DDclient-Gandi | DNS update | - |
| Cloudflare DDNS | CF update | - |
| Speedtest Tracker | Speed test | 8765 |
| Netmaker | Mesh VPN | 51821/51820 |

### Installation

```bash
# Pi-hole
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/pihole.sh)"

# AdGuard Home
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/adguard.sh)"

# WireGuard
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/wireguard.sh)"

# Tailscale
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/tailscale.sh)"
```

---

## Monitoring & Analytics

### Metrics & Dashboards

| Script | Description | Ports |
|-------|-------------|------|
| Grafana | Metrics dashboard | 3000 |
| Prometheus | Metrics collection | 9090 |
| Netdata | Real-time monitoring | 19999 |
| Zabbix | Enterprise monitoring | 80/10051 |
| Uptime Kuma | Uptime monitor | 3001 |
| Statping | Status page | 8080 |
| Checkmk | Monitoring | 5000 |

### Logging

| Script | Description | Ports |
|-------|-------------|------|
| Glances | System monitor | 61208 |
| GoAccess | Log analyzer | 7890 |
| Dozzle | Docker logs | 8080 |
| Jaeger | Tracing | 16686 |

### Installation

```bash
# Uptime Kuma
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/kuma.sh)"

# Netdata
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/netdata.sh)"

# Grafana
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/grafana.sh)"

# Prometheus
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/prometheus.sh)"
```

---

## Databases

### Relational Databases

| Script | Description | Ports |
|-------|-------------|------|
| PostgreSQL | RDBMS | 5432 |
| MariaDB | MySQL fork | 3306 |
| MySQL | RDBMS | 3306 |

### NoSQL

| Script | Description | Ports |
|-------|-------------|------|
| MongoDB | Document DB | 27017 |
| Redis | In-memory | 6379 |

### Time Series

| Script | Description | Ports |
|-------|-------------|------|
| InfluxDB | Time series DB | 8086 |
| QuestDB | Time series | 9000 |
| TimescaleDB | PostgreSQL ext | 5432 |

### Installation

```bash
# PostgreSQL
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/postgres.sh)"

# MariaDB
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/mariadb.sh)"

# Redis
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/redis.sh)"

# InfluxDB
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/influxdb.sh)"
```

---

## Security

### Authentication

| Script | Description | Ports |
|-------|-------------|------|
| Authelia | 2FA auth | 9091 |
| Authentik | Identity provider | 9000 |
| Keycloak | Identity manager | 8080 |
| OPA | Policy engine | 8181 |

### Password Management

| Script | Description | Ports |
|-------|-------------|------|
| Vaultwarden | Password manager | 80 |
| Bitwarden | Password manager | 80 |
| Passbolt | Password manager | 443 |
| Padloc | Password manager | 3000 |

### Security Tools

| Script | Description | Ports |
|-------|-------------|------|
| CrowdSec | Crowd-sourced security | 8080 |
| Suricata | IDS/IPS | 9000 |
| Wazuh | SIEM | 55000 |
| Trivy | Vulnerability scanner | 8080 |
| ClamAV | Antivirus | 3310 |

### Installation

```bash
# Vaultwarden
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/vaultwarden.sh)"

# Authelia
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/authelia.sh)"

# Authentik
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/authentik.sh)"

# CrowdSec
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/crowdsec.sh)"
```

---

## Development Tools

### Code Management

| Script | Description | Ports |
|-------|-------------|------|
| Gitea | Git service | 3000 |
| GitLab | Dev platform | 80/443 |
| Forgejo | Git service | 3000 |

### Dev Environments

| Script | Description | Ports |
|-------|-------------|------|
| VS Code Server | Browser IDE | 8443 |
| code-server | Browser IDE | 8443 |
| Coder | Dev environment | 8443 |
| Ninja | IDE | 8080 |

### Automation

| Script | Description | Ports |
|-------|-------------|------|
| n8n | Workflow automation | 5678 |
| Pipedream | Workflow | 3000 |
| PipeRage | Workflow | 8080 |

### Container Tools

| Script | Description | Ports |
|-------|-------------|------|
| Portainer | Container UI | 9000 |
| Yacht | Container manager | 8000 |
| Lazydocker | TUI docker | - |
| Dozzle | Docker logs | 8080 |

### Package Management

| Script | Description | Ports |
|-------|-------------|------|
| JFrog | Artifact manager | 8082 |
| Verdaccio | NPM registry | 4873 |
| Nexus | Artifact manager | 8081 |

### Installation

```bash
# Gitea
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/gitea.sh)"

# VS Code Server
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/codeserver.sh)"

# n8n
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/n8n.sh)"

# Portainer
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/portainer.sh)"
```

---

## Documents & Notes

### Document Management

| Script | Description | Ports |
|-------|-------------|------|
| Paperless | Document mgmt | 8000 |
| Docsumo | Document OCR | 8000 |
| внутр | Doc management | 8000 |

### Wikis & Notes

| Script | Description | Ports |
|-------|-------------|------|
| WikiJS | Wiki platform | 3000 |
| Outline | Wiki/notes | 3000 |
| Bookstack | Wiki | 80 |
| DokuWiki | Wiki | 80 |
| HackMD | Markdown notes | 3000 |
| Joplin | Notes | 22300 |

### Installation

```bash
# Paperless-NGX
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/paperless.sh)"

# WikiJS
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/wikijs.sh)"

# Bookstack
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/bookstack.sh)"
```

---

## MQTT & Messaging

### IoT Messaging

| Script | Description | Ports |
|-------|-------------|------|
| MQTT | Message broker | 1883/9001 |
| EMQX | Enterprise MQTT | 18083 |
| VerneMQ | MQTT broker | 1883 |
| Mosquitto | MQTT broker | 1883 |

### Chat & Notification

| Script | Description | Ports |
|-------|-------------|------|
| Matrix | Chat server | 8008 |
| Synapse | Matrix server | 8008 |
| Modlast | Matrix client | 3000 |
| Mattermost | Team chat | 8065 |

### Installation

```bash
# Mosquitto
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/mosquitto.sh)"

# EMQX
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/emqx.sh)"

# Synapse
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/synapse.sh)"
```

---

## Automation & Scheduling

### Workflow Automation

| Script | Description | Ports |
|-------|-------------|------|
| n8n | Workflow | 5678 |
| PipeRage | Automation | 8080 |
| Pipedream | Workflow | 3000 |
| I-Feel | Automation | 3000 |

### Cron & Scheduling

| Script | Description | Ports |
|-------|-------------|------|
| Cronicle | Job scheduler | 3002 |
| CronMaster | Cron UI | 3000 |

### Installation

```bash
# Cronicle
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/cronicle.sh)"

# CronMaster
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/cronmaster.sh)"
```

---

## NVR & Cameras

### Camera Systems

| Script | Description | Ports |
|-------|-------------|------|
| Frigate | NVR | 5000 |
| MotionEye | Camera NVR | 8765 |
| Shinobi | NVR | 8080 |
| ZoneMinder | NVR | 80/443 |

### Installation

```bash
# Frigate
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/frigate.sh)"

# MotionEye
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/motioneye.sh)"

# Shinobi
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/shinobi.sh)"
```

---

## Webservers & Proxies

### Reverse Proxies

| Script | Description | Ports |
|-------|-------------|------|
| Nginx Proxy Manager | Reverse proxy | 81/444 |
| Traefik | Reverse proxy | 80/443 |
| Caddy | Web server | 80/443 |
| Apache | Web server | 80 |

### Installation

```bash
# Nginx Proxy Manager
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/npm.sh)"

# Traefik
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/traefik.sh)"
```

---

## Proxmox & Virtualization

### PVE Management Scripts

| Script | Description |
|--------|-------------|
| PVE LXC Execute | Run commands in LXCs |
| PVE Startup Dependency Check | Check VM dependencies |
| PVE Update Repositories | Update repos |
| PVE LXC Updater | Update containers |
| PVE Scripts Local | Local script manager |
| CronMaster | Cron job scheduler |

### Installation

```bash
# PVE Scripts Local (adds menu to Proxmox)
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/pve-scripts-local.sh)"
```

### PVE Scripts

#### LXC Execute

```bash
# Execute command in all/specific containers
# Available in PVE Scripts Local menu
```

#### Startup Dependency Check

```bash
# Ensures storage availability before VM start
# Adds dependency ordering
```

#### All Templates

```bash
# Create system LXC templates
# Creates *.creds file with password
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/all-templates.sh)"
```

---

## Miscellaneous

### System Administration

| Script | Description | Ports |
|-------|-------------|------|
| Webmin | Sys admin | 10000 |
| Cockpit | Sys admin | 9090 |
| Proxmox GUI | PMG/TrueNAS | - |

### File Management

| Script | Description | Ports |
|-------|-------------|------|
| FileBrowser | File manager | 8080 |
| Nextcloud | File sync | 80/443 |
| OwnCloud | File sync | 80 |
| Syncthing | File sync | 8384 |

### Download Tools

| Script | Description | Ports |
|-------|-------------|------|
| qBittorrent | BT downloader | 8080 |
| qBittorrent-nox | BT CLI | 8080 |
| Transmission | BT downloader | 9091 |
| SABnzbd | Usenet | 8080 |

### Other Tools

| Script | Description | Ports |
|-------|-------------|------|
| Calibre | E-books | 8083 |
| Calibre Web | E-books | 8083 |
| Ubooquity | Comics | 8787 |
| Lidarr | Music | 8686 |
| SoundStorm | Music | 8380 |

### Installation

```bash
# Webmin
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/webmin.sh)"

# FileBrowser
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/filebrowser.sh)"

# qBittorrent
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/qbittorrent.sh)"
```

---

## Script Management

### Update Scripts

```bash
# Update all community-scripts containers
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/pve-lxc-apps.sh)"

# Update PVE
apt update && apt upgrade -y
```

### Common Options

Most scripts ask for:
- **Container ID** (or auto-assign)
- **Password** (for services)
- **Domain** (optional)
- **Enable Docker** (some scripts)
- **Advanced options** (optional)

---

## Function Libraries

### Core Libraries

| Library | Purpose |
|---------|---------|
| `build.func` | Container build functions |
| `install.func` | Installation helpers |
| `tools.func` | Utility functions |
| `core.func` | Core PVE functions |
| `error_handler.func` | Error handling |

### Usage in Scripts

```bash
# Source libraries
source /func/${lib}.func

# Use functions
$STDYAN "Installing..."
set -e
```

---

## Quick Reference

### Top 20 Scripts

| # | Script | Install Command |
|----|--------|---------------|
| 1 | Docker | `docker.sh` |
| 2 | Portainer | `portainer.sh` |
| 3 | AdGuard | `adguard.sh` |
| 4 | Pi-hole | `pihole.sh` |
| 5 | Home Assistant | `homeassistant.sh` |
| 6 | Jellyfin | `jellyfin.sh` |
| 7 | Radarr | `radarr.sh` |
| 8 | Sonarr | `sonarr.sh` |
| 9 | NPM | `npm.sh` |
| 10 | Vaultwarden | `vaultwarden.sh` |
| 11 | WireGuard | `wireguard.sh` |
| 12 | Uptime Kuma | `kuma.sh` |
| 13 | Traefik | `traefik.sh` |
| 14 | VS Code | `codeserver.sh` |
| 15 | n8n | `n8n.sh` |
| 16 | Gitea | `gitea.sh` |
| 17 | Nginx | `nginx.sh` |
| 18 | Netdata | `netdata.sh` |
| 19 | MariaDB | `mariadb.sh` |
| 20 | PostgreSQL | `postgres.sh` |

---

## Links

- **Website**: https://community-scripts.org/
- **GitHub**: https://github.com/community-scripts/ProxmoxVE
- **Docs**: https://community-scripts.github.io/ProxmoxVE/
- **PVE-Local**: https://github.com/community-scripts/ProxmoxVE-Local

## Keywords

#community-scripts #proxmox #scripts #automation #homelab #containers #docker #selfhosted #homelautomation

## Related

- [[Proxmox-VE]] - Main documentation
- [[HomeLab]] - Home lab setup
- [[Scripts]] - Custom scripts
- [[Containers]] - LXC setup
- [[VM]] - VM setup
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
