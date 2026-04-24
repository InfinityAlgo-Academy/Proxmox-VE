# Proxmox Home Lab

## Table of Contents

1. [Home Lab Overview](#home-lab-overview)
2. [Network Design](#network-design)
3. [Hardware Recommendations](#hardware-recommendations)
4. [VM/Container Layout](#vmcontainer-layout)
5. [Common Services](#common-services)
6. [Security Considerations](#security-considerations)
7. [Backup Strategy](#backup-strategy)
8. [Monitoring](#monitoring)
9. [Performance Tuning](#performance-tuning)
10. [Maintenance](#maintenance)

---

## Home Lab Overview

### What is a Home Lab?

A home lab is a personal computing environment for:
- Learning new technologies
- Testing software
- Self-hosting services
- Running home automation
- Development environments

### Proxmox Advantages for Home Lab

- **Free**: No licensing costs
- **Flexible**: Mix of VMs and containers
- **Easy**: Web-based management
- **Powerful**: Enterprise features
- **Learning**: Real-world virtualization skills

### Use Cases

| Use Case | Services |
|----------|----------|
| Home Automation | Home Assistant, MQTT, Node-RED |
| Media | Jellyfin, Radarr, Sonarr, Plex |
| Downloads | qBittorrent, NZBGet, Transmission |
| Network | Pi-hole, AdGuard, pfSense |
| Development | Docker, code servers, databases |
| Security | Vaultwarden, WireGuard, crowdsec |
| Finance | Firefly III, Budget Zero |
| Smart Home | Home Assistant, zigbee2mqtt |

---

## Network Design

### Basic Home Lab Network

```
Internet
  │
  ▼
┌─────────────────────────┐
│      Router/Modem      │
│    192.168.1.1/24    │
└──────────┬────────────┘
           │
           │ (WAN)
           ▼
┌─────────────────────────┐
│    Proxmox Host        │
│    192.168.1.100      │
│         pve1           │
└──────────┬────────────┘
           │
    ┌──────┴──────┐
    │             │
┌───▼────┐  ┌───▼────┐
│ vmbr0  │  │ vmbr1  │
│ VMs   │  │ Internal│
│ LAN  │  │   Isolated│
│1.100 │  │ 10.0.1 │
└──────┘  └────────┘
```

### VLAN Setup

| VLAN | Network | Purpose |
|------|---------|---------|
| 1 | 192.168.1.0/24 | Primary LAN |
| 10 | 192.168.10.0/24 | IoT Devices |
| 20 | 192.168.20.0/24 | Guest Network |
| 30 | 192.168.30.0/24 | Management |
| 40 | 192.168.40.0/24 | Storage |

### DNS Schema

| Service | IP | Hostname |
|--------|-----|----------|
| Proxmox | 192.168.1.100 | pve.local |
| Pi-hole | 192.168.1.50 | pihole.local |
| Home Assistant | 192.168.1.51 | homeassistant.local |
| Jellyfin | 192.168.1.52 | jellyfin.local |
| AdGuard | 192.168.1.53 | adguard.local |
| Vaultwarden | 192.168.1.54 | vault.local |

### Recommended Services

```bash
# DNS/DHCP (Pi-hole)
# Container
pct create 200 \
  local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --cores 1 --memory 512 \
  --hostname pihole \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.50/24,gw=192.168.1.1 \
  --dns 1.1.1.1 --search local
```

---

## Hardware Recommendations

### Minimum Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4GB | 8-16GB |
| System Disk | 32GB SSD | 64GB+ SSD |
| Data Disk | 500GB | 1TB+ HDD/SSD |
| Network | 1GbE | 1GbE+ |

### Hardware Options

#### Budget Build
- **CPU**: Intel i3-6100 / AMD Ryzen 3
- **RAM**: 8GB (2x4GB)
- **Storage**: 120GB SSD + 1TB HDD
- ** NIC: Onboard

#### Mid-Range Build
- **CPU**: Intel i5-7500 / AMD Ryzen 5
- **RAM**: 16GB (2x8GB)
- **Storage**: 256GB NVMe + 2TB HDD
- ** NIC**: Dual NIC

#### High-End Build
- **CPU**: Intel i7-9700 / AMD Ryzen 7
- **RAM**: 32GB (2x16GB)
- **Storage**: 512GB NVMe (VM) + 4TB HDD (data) + 2TB SSD (cache)
- ** NIC**: 10GbE (optional)

### Server Recommendations

| Server Model | CPU | RAM Slots | Drive Bays |
|-------------|-----|----------|------------|
| Dell OptiPlex 7080 | i7 | 4 | 2x3.5" |
| HP ProDesk 600 | i7 | 4 | 2x3.5" |
| Lenovo M920 | i7 | 4 | 2x3.5" |
| R720 | E5-2600v2 | 8 | 12x3.5" |

---

## VM/Container Layout

### Recommended Allocation

#### VM (Virtual Machines)

| VM ID | Name | OS | vCPU | RAM | Disk | Purpose |
|------|------|-----|-----|-----|------|--------|
| 100 | pfSense | pfSense | 2 | 2GB | 8GB | Firewall |
| 101 | Windows | Win11 | 2 | 4GB | 64GB | Windows VM |
| 102 | Home Assistant | HA OS | 2 | 4GB | 32GB | Smart Home |

#### CT (Containers)

| CT ID | Name | OS | RAM | Disk | Purpose |
|------|------|-----|-----|------|------|
| 200 | Pi-hole | Debian | 512MB | 2GB | DNS/DHCP |
| 201 | AdGuard | Debian | 512MB | 2GB | Ad Blocker |
| 202 | Vaultwarden | Debian | 512MB | 2GB | Password Manager |
| 203 | Nginx Proxy | Debian | 256MB | 1GB | Reverse Proxy |
| 204 | MariaDB | Debian | 1GB | 10GB | Database |
| 205 | Jellyfin | Debian | 2GB | 50GB | Media Server |
| 206 | qBittorrent | Debian | 1GB | 10GB | Downloads |

### Resource Planning

```
Total RAM: 16GB

System: 2GB
VMs:
  pfSense: 2GB
  Home Assistant: 4GB
  Windows: 4GB
Containers: ~6GB available

Storage:
  System: 32GB
  VMs + CTs: 100GB
  Media: Remaining
```

---

## Common Services

### Network Services

#### Pi-hole

```bash
# Install Pi-hole
pct create 200 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname pihole \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && curl -sL https://install.pi-hole.net | bash

# Set static IP post-install
```

#### AdGuard Home

```bash
# Install AdGuard
pct create 201 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname adguard \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && apt install -y curl
curl -sfL https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v

# Web UI: http://<IP>:3000
```

#### pfSense

```bash
# Install pfSense
qm create 100 \
  --name pfsense \
  --memory 2048 \
  --cores 2 \
  --net0 e1000,bridge=vmbr0 \
  --net1 e1000,bridge=vmbr1 \
  --net2 e1000,bridge=vmbr2 \
  --cdrom local:iso/pfsense.iso \
  --boot order=ide2

# Configure via console after install
```

### Home Automation

#### Home Assistant Container

```bash
# Install Home Assistant
pct create 202 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:8 \
  --hostname homeassistant \
  --memory 4096 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1

# In container
apt update
apt install -y python3 python3-pip docker.io
usermod -aG docker root
systemctl enable docker
systemctl start docker

# Run HA
docker run -d \
  --name homeassistant \
  --restart unless-stopped \
  -v /homeassistant/config:/config \
  --network=host \
  homeassistant/home-assistant:stable
```

#### zigbee2mqtt

```bash
# Install zigbee2mqtt
pct create 203 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname zigbee2mqtt \
  --memory 256 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && apt install -y nodejs npm git
git clone https://github.com/Koenkk/zigbee2mqtt.git /opt/zigbee2mqtt
cd /opt/zigbee2mqtt
npm install

# Configure serial port passthrough
# Add USB device to container
pct set 203 --usb0 host=10c4:ea60
```

### Media Services

#### Jellyfin

```bash
# Install Jellyfin
pct create 205 local:vztmpl/ubuntu-22.04-standard_22.04.tar.gz \
  --rootfs local:50 \
  --hostname jellyfin \
  --memory 2048 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && apt install -y apt-transport-https
wget -O- https://repo.jellyfin.org/jellyfin_team.gpg.key | apt-key add -
echo "deb https://repo.jellyfin.org/ubuntu focal main" | tee /etc/apt/sources.list.d/jellyfin.list
apt update && apt install -y jellyfin

systemctl enable jellyfin
systemctl start jellyfin

# Web UI: http://<IP>:8096
```

#### Radarr/Sonarr

```bash
# Install Radarr
pct create 206 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:10 \
  --hostname radarr \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && apt install -y mono-utils libsqlite3-dev
wget https://radarr.servarr.com/v2/download/develop/latest -O /opt/radarr.tar.gz
tar -xzf /opt/radarr.tar.gz -C /opt/
rm /opt/radarr.tar.gz

# Run
cd /opt/Radarr && mono ./Radarr.exe
```

### Security

#### Vaultwarden

```bash
# Install Vaultwarden
pct create 202 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:2 \
  --hostname vaultwarden \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && apt install -y docker.io docker-compose
usermod -aG docker root

# Run Vaultwarden
docker run -d \
  --name=vaultwarden \
  --restart=always \
  -v /vw-data:/data \
  -p 80:80 \
  -e SIGNUP_ALLOWED=true \
  vaultwarden/server:latest

# Web UI: http://<IP>:80
```

#### WireGuard

```bash
# Install WireGuard
pct create 207 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:2 \
  --hostname wireguard \
  --memory 256 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && apt install -y wireguard wireguard-tools

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey
wg genkey | tee server-privatekey | wg pubkey > server-publickey

# Configure /etc/wireguard/wg0.conf
[Interface]
Address = 10.0.0.1/24
PrivateKey = <server-privatekey>
ListenPort = 51820

[Peer]
PublicKey = <client-publickey>
AllowedIPs = 10.0.0.2/32

# Enable
wg-quick up wg0
systemctl enable wg-quick@wg0
```

### Reverse Proxy

#### Nginx Proxy Manager

```bash
# Install NPM
pct create 203 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname nginx-proxy \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# In container
apt update && apt install -y docker.io docker-compose
git clone https://github.com/NginxProxyManager/nginx-proxy-manager.git /opt/npm
cd /opt/npm
docker-compose up -d

# Web UI: http://<IP>:81
# Default: admin@example.com / changeme
```

### Development

#### Docker Host

```bash
# Create Docker host
pct create 204 local:vztmpl/ubuntu-22.04-standard_22.04.tar.gz \
  --rootfs local:20 \
  --hostname docker \
  --memory 4096 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1 \
  --features keyctl=1

# In container
apt update && apt install -y docker.io docker-compose
usermod -aG docker root
systemctl enable docker
systemctl start docker
```

---

## Security Considerations

### Network Isolation

```bash
# Create isolated bridge for IoT
auto vmbr10
iface vmbr10 inet manual
    bridge-ports none
    bridge-stp off
```

### Firewall Rules

```bash
# Allow only necessary ports
ufw default deny incoming
ufw allow 8006/tcp from 192.168.1.0/24
ufw allow 22/tcp from 192.168.1.0/24
ufw enable
```

### VPN Access

```bash
# Only access via VPN
# Block external access to services
# Use WireGuard for remote access
```

### Container Security

- Use unprivileged containers
- Keep containers updated
- Review permissions regularly
- Use strong passwords

---

## Backup Strategy

### Local Backup

```bash
# Backup all VMs and CTs
vzdump all --mode snapshot --storage local-backup --compress lzo
```

### Offsite Backup

```bash
# Sync to remote
rsync -avz /mnt/local-backup/dump/ user@backup-server:/backup/

# Or to cloud
rclone sync /mnt/local-backup/dump/ gdrive:ProxmoxBackup
```

### Backup Schedule

| Data Type | Frequency | Retention |
|-----------|-----------|-----------|
| Full System | Monthly | 12 months |
| VMs | Weekly | 4 weeks |
| Config | Daily | 7 days |
| Media | None | - |

---

## Monitoring

### Basic Monitoring

```bash
# Resource check script
# Run via cron
# Alert on high usage
```

### Grafana + Prometheus

```bash
# Install for detailed monitoring
# Track VM/CT resource usage over time
# Create dashboards
```

### Notifications

```bash
# Set up notifications
# Email or Discord/Slack webhooks
```

---

## Performance Tuning

### Container Optimization

```bash
# Use unprivileged containers
# Limit resources properly
# Use VirtIO drivers in VMs
```

### Storage Optimization

```bash
# Use SSD for VM storage
# Enable discard
# Use proper cache modes
```

### Network Optimization

```bash
# Use VirtIO network drivers
# Enable multi-queue
# Enable jumbo frames where supported
```

---

## Maintenance

### Regular Tasks

| Task | Frequency |
|------|-----------|
| Update system | Weekly |
| Check logs | Daily |
| Verify backups | Weekly |
| Clean old backups | Monthly |
| Review resources | Weekly |

### Update Process

```bash
# Update Proxmox
apt update && apt upgrade -y
reboot

# Update containers
pct exec <ctid> -- bash -c "apt update && apt upgrade -y"
```

---

## Keywords

#homelab #selfhosted #homeautomation #services #network #media #security #development #monitoring

## Links

- [[Proxmox-VE]] - Main documentation
- [[Installation]] - Installation guide
- [[Containers]] - Container setup
- [[VM]] - VM setup
- [[Networking]] - Network setup
- [[Security]] - Security hardening
- [[Backup]] - Backup strategy
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
