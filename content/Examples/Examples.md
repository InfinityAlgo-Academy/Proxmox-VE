# Proxmox VE Examples - Practical Setups

## Table of Contents

1. [Home Lab Example](#home-lab-example)
2. [Development Environment](#development-environment)
3. [Media Server Setup](#media-server-setup)
4. [Home Automation Stack](#home-automation-stack)
5. [Security Stack](#security-stack)
6. [Development Stack](#development-stack)
7. [Network Stack](#network-stack)
8. [Backup Solutions](#backup-solutions)
9. [Monitoring Stack](#monitoring-stack)

---

## Home Lab Example

### Network Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                  Home Lab Network                       │
├─────────────────────────────────────────────────────────────┤
│ Internet                                               │
│   │                                                    │
│   ▼ (WAN)                                             │
│ ┌──────────┐                                           │
│ │ Router  │ ── DHCP: 192.168.1.1                         │
│ └──────────┘                                           │
│   │                                                    │
│   │ (Bridge)                                          │
│   ▼                                                    │
│ ┌──────────────────────────────────────┐            │
│ │ Proxmox Host (192.168.1.100)            │            │
│ │                                      │            │
│ │ ├─ vmbr0: 192.168.1.0/24 (LAN)      │            │
│ │ └─ vmbr1: 10.0.10.0/24 (Internal)│            │
│ └──────────────────────────────────────┘            │
│        │                                               │
│        ├── VMs                                         │
│        │  ├─ pfSense (VM 100)                       │
│        │  └─ Windows 11 (VM 101)                       │
│        │                                              │
│        └── Containers                                 │
│           ├─ Pi-hole (CT 200)                      │
│           ├─ Portainer (CT 201)                   │
│           ├─ AdGuard (CT 202)                      │
│           ├─ WireGuard (CT 203)                   │
│           └─ Jellyfin (CT 204)                      │
└─────────────────────────────────────────────────────────────┘
```

### IP Allocation

| Service | Type | IP | CTID/VMID |
|---------|------|-----|----------|
| Proxmox | Host | 192.168.1.100 | - |
| Router | Physical | 192.168.1.1 | - |
| pfSense | VM | 192.168.1.254 | 100 |
| Pi-hole | CT | 192.168.1.50 | 200 |
| AdGuard | CT | 192.168.1.51 | 202 |
| Jellyfin | CT | 192.168.1.52 | 204 |
| Portainer | CT | 201 | - |
| WireGuard | CT | 203 | - |

### Implementation

```bash
# 1. Create Networks
# VM Network (vmbr0) already exists

# Create internal network
auto vmbr1
iface vmbr1 inet static
    address 10.0.10.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

# 2. Create Containers
# Pi-hole
pct create 200 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname pihole \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.50/24,gw=192.168.1.1 \
  --dns 1.1.1.1

# 3. Install Pi-hole
pct exec 200 -- bash -c "curl -sL https://install.pi-hole.net | bash"

# 4. Configure Router DNS to point to Pi-hole
# Login to router and set DNS to 192.168.1.50
```

---

## Development Environment

### Setup

```bash
# Create development container
pct create 210 local:vztmpl/ubuntu-22.04-standard_22.04.tar.gz \
  --rootfs local:30 \
  --hostname devbox \
  --memory 8192 --cores 4 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1,keyctl=1,fuse=1 \
  --onboot 1

# Install development tools
pct exec 210 -- bash -c "
apt update && apt upgrade -y
apt install -y build-essential git curl wget vim \
  nodejs npm docker.io python3 python3-pip \
  golang-go openjdk-17-jdk
"
```

### Installed Tools

- **Git** - Version control
- **Docker** - Container runtime
- **Node.js** - JavaScript runtime
- **Python** - Python development
- **Golang** - Go development
- **Java JDK** - Java development

### VS Code Server

```bash
# Install in development container
pct exec 210 -- bash -c "
curl -fsSL https://code-server.dev/install.sh | sh
systemctl enable code-server@root
systemctl start code-server@root
"

# Access at http://192.168.1.x:8443
```

---

## Media Server Setup

### Media Server Stack

```bash
# Create Jellyfin
pct create 220 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:100 \
  --hostname jellyfin \
  --memory 4096 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Install Jellyfin
pct exec 220 -- bash -c "
apt update
wget -O- https://repo.jellyfin.org/jellyfin_team.gpg.key | apt-key add -
echo 'deb https://repo.jellyfin.org/ubuntu bookworm main' > /etc/apt/sources.list.d/jellyfin.list
apt update
apt install -y jellyfin
systemctl enable jellyfin
systemctl start jellyfin
"
```

### Media Tools Stack

```bash
# Radarr
pct create 221 --rootfs local:10 \
  --hostname radarr --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Sonarr
pct create 222 --rootfs local:10 \
  --hostname sonarr --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# qBittorrent
pct create 223 --rootfs local:20 \
  --hostname qbittorrent --memory 1024 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp
```

### Storage Setup

```bash
# Add storage for media
pvesm add dir media-nfs \
  --server 192.168.1.254 \
  --export /mnt/media \
  --content rootdir,images \
  --mount-options rsize=8192,wsize=8192

# Mount as bind mount to container
pct set 220 --mp0 /mnt/media,mp=/media
```

---

## Home Automation Stack

### Home Assistant

```bash
# Create Home Assistant
pct create 230 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:8 \
  --hostname homeassistant \
  --memory 4096 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1

# Run Home Assistant
docker run -d --name homeassistant \
  --restart unless-stopped \
  --network host \
  -v /opt/homeassistant/config:/config \
  --device /dev/ttyUSB0:/dev/ttyUSB0 \
  homeassistant/home-assistant:latest
```

### Zigbee2MQTT

```bash
# Create Zigbee2MQTT
pct create 231 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname zigbee2mqtt \
  --memory 256 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Install and run
pct exec 231 --bash -c "
apt update && apt install -y nodejs npm git
git clone https://github.com/Koenkk/zigbee2mqtt.git /opt/zigbee2mqtt
cd /opt/zigbee2mqtt
npm install
npm start
"

# Add USB passthrough
pct set 231 --usb0 host=10c4:ea60
```

### ESPHome

```bash
# Create ESPHome
pct create 232 local:vztmpl/python3.tar.gz \
  --rootfs local:4 \
  --hostname esphome \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Install ESPHome
pct exec 232 -- pip3 install esphome
pct exec 232 -- esphome config /config
```

---

## Security Stack

### VPN (WireGuard)

```bash
# Create WireGuard
pct create 240 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:2 \
  --hostname wireguard \
  --memory 256 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Install WireGuard
pct exec 240 -- bash -c "
apt update && apt install -y wireguard wireguard-tools wg-quick

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Configure
cat > /etc/wireguard/wg0.conf '[Interface]
Address = 10.0.0.1/24
PrivateKey = <private-key>
ListenPort = 51820

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32'

wg-quick up wg0
systemctl enable wg-quick@wg0
"
```

### Password Manager (Vaultwarden)

```bash
# Create Vaultwarden
pct create 241 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:2 \
  --hostname vaultwarden \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Run Vaultwarden
pct exec 241 -- docker run -d \
  --name vaultwarden \
  -v /opt/vaultwarden:/data \
  -p 80:80 \
  --restart unless-stopped \
  vaultwarden/server:latest
```

### 2FA (Authelia)

```bash
# Create Authelia
pct create 242 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname authelia \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Run Authelia
pct exec 242 -- docker run -d \
  --name authelia \
  -v /opt/authelia:/config \
  -p 9091:9091 \
  --restart unless-stopped \
  authelia/authelia:latest
```

---

## Development Stack

### Git Server (Gitea)

```bash
# Create Gitea
pct create 250 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:20 \
  --hostname gitea \
  --memory 2048 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1

# Install Gitea
pct exec 250 -- bash -c "
mkdir -p /opt/gitea
cd /opt/gitea
curl -sL https://github.com/go-gitea/gitea/releases/download/v1.21.0/gitea-1.21.0-linux-amd64 -o gitea
chmod +x gitea
mkdir -p /opt/gitea/custom /opt/gitea/data /opt/gitea/log

useradd -m -s /bin/bash gitea
chown -R gitea:gitea /opt/gitea

sudo -u gitea ./gitea web -c custom/conf/app.ini
"
```

### CI/CD (GitLab)

```bash
# For GitLab, use VM (requires more resources)
qm create 101 \
  --name gitlab \
  --memory 16384 \
  --cores 8 \
  --cpu host \
  --scsi0 local:vm-101-disk-0,size=250G \
  --net0 virtio,bridge=vmbr0 \
  --cdrom local:iso/gitlab.iso

# Or use community-scripts
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/gitlab.sh)"
```

### n8n Workflow

```bash
# Create n8n
pct create 251 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:10 \
  --hostname n8n \
  --memory 1024 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1

# Run n8n
pct exec 251 -- docker run -d \
  --name n8n \
  -v /opt/n8n:/home/node/.n8n \
  -p 5678:5678 \
  --restart unless-stopped \
  n8nio/n8n:latest
```

---

## Network Stack

### pfSense

```bash
# Create pfSense VM
qm create 100 \
  --name pfsense \
  --memory 2048 \
  --cores 2 \
  --cpu host \
  --net0 e1000,bridge=vmbr0 \
  --net1 e1000,bridge=vmbr1 \
  --net2 e1000,bridge=vmbr2 \
  --scsi0 local:vm-100-disk-0,size=8G \
  --cdrom local:iso/pfsense.iso \
  --boot order=ide2

# Install pfSense via console
# WAN: vmbr0, LAN: vmbr1, OPT1: vmbr2
```

### Reverse Proxy (Nginx Proxy Manager)

```bash
# Create NPM
pct create 260 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname npm \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Run NPM
pct exec 260 -- docker run -d \
  --name npm \
  -p 80:80 -p 443:443 -p 81:81 \
  -v npm-data:/data \
  --restart unless-stopped \
  jc21/nginx-proxy-manager:latest
```

### Pi-hole + unbound

```bash
# Create Pi-hole with unbound
pct create 270 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname pihole-unbound \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.50/24,gw=192.168.1.1

# Install Pi-hole + unbound
pct exec 270 -- bash -c "
curl -sL https://install.pi-hole.net | bash

# Add unbound DNS
apt install -y unbound

cat > /etc/unbound/unbound.conf.d/pi-hole.conf << 'EOF'
server:
    port: 5353
    bind-interfaces: true
    private-domain: local
    domain: local
    cache-size: 10000
EOF

systemctl restart unbound
"
```

---

## Backup Solutions

### Local Backup

```bash
# Backup script
#!/bin/bash
# /root/scripts/backup.sh

STORAGE="local"
DATE=$(date +%Y%m%d)

# Backup VMs
for vm in $(qm list | awk 'NR>1 {print $1}' | tr -d '*'); do
    qmid=$(echo $vm | tr -d '*')
    [ -z "$qmid" ] && continue
    echo "Backing up VM $qmid..."
    vzdump $qmid --mode snapshot --storage $STORAGE --compress lzo
done

# Backup CTs
for ct in $(pct list | awk 'NR>1 {print $1}'); do
    [ -z "$ct" ] && continue
    echo "Backing up CT $ct..."
    vzdump $ct --mode snapshot --storage $STORAGE --compress lzo
done

echo "Backup complete: $DATE"
```

### rsync Backup

```bash
# rsync to remote
rsync -avz --delete \
  /mnt/local-backup/dump/ \
  user@backup-server:/backup/proxmox/
```

---

## Monitoring Stack

### Grafana + Prometheus

```bash
# Prometheus
pct create 280 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:10 \
  --hostname prometheus \
  --memory 2048 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

pct exec 280 -- docker run -d \
  --name prometheus \
  -v prometheus-data:/prometheus \
  -p 9090:9090 \
  --restart always \
  -f /etc/prometheus/prometheus.yml \
  prom/prometheus:latest

# Grafana
pct create 281 --rootfs local:5 \
  --hostname grafana --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

pct exec 281 -- docker run -d \
  --name grafana \
  -v grafana-data:/var/lib/grafana \
  -p 3000:3000 \
  --restart always \
  grafana/grafana
```

### Uptime Kuma

```bash
# Uptime Kuma
pct create 282 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:5 \
  --hostname kuma \
  --memory 512 --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1

pct exec 282 -- docker run -d \
  --name kuma \
  -v kuma-data:/app/data \
  -p 3001:3001 \
  --restart always \
  louislam/uptime-kuma:latest
```

---

## Quick Reference

### All-in-One Install Scripts

```bash
# Docker
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)"

# Portainer
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/portainer.sh)"

# Home Assistant
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/homeassistant.sh)"

# Traefik
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/traefik.sh)"
```

## Keywords

#examples #practical #homelab #stack #setup #configuration

## Links

- [[Proxmox-VE]] - Main documentation
- [[HomeLab]] - Home lab overview
- [[Community-Scripts]] - One-click scripts
- [[Docker]] - Docker guide
---

[[index|Back to Proxmox VE]]
