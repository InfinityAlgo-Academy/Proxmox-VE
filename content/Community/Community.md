---
title: Proxmox Community Scripts & Guides
---

# Proxmox Community Scripts & Guides

> **For complete community-scripts documentation, see**: [[Community-Scripts]]

## Quick Reference

Quick install commands for popular services:

```bash
# Docker
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)"

# Portainer
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/portainer.sh)"

# Home Assistant
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/homeassistant.sh)"

# Pi-hole
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/pihole.sh)"

# AdGuard
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/adguard.sh)"

# Jellyfin
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/jellyfin.sh)"
```

---

## Table of Contents

1. [Community Resources](#community-resources)
2. [Popular Scripts](#popular-scripts)
3. [Helper Scripts](#helper-scripts)
4. [Home Lab Scripts](#home-lab-scripts)
5. [Container Templates](#container-templates)
6. [VM Optimizations](#vm-optimizations)
7. [Networking Scripts](#networking-scripts)
8. [Storage Scripts](#storage-scripts)
9. [Backup Solutions](#backup-solutions)
10. [Monitoring Solutions](#monitoring-solutions)

---

## Community Resources

### Official Forum

- **Forum**: https://forum.proxmox.com/
- **Wiki**: https://pve.proxmox.com/wiki/
- **Bug Tracker**: https://bugzilla.proxmox.com/

### Community Contributors

| Author | Scripts | Link |
|--------|---------|------|
| Timothy | All-in-one scripts | https://tteck.github.io/Proxmom |
| Community | Helper scripts | Various |
| Users | Custom configs | Forum |

### Useful Links

- Proxmox Wiki: https://pve.proxmox.com/wiki/
- Reddit r/Proxmox: https://reddit.com/r/Proxmox
- GitHub Topics: https://github.com/topics/proxmox

---

## Popular Scripts

### Proxmox VE Helper Scripts (tteck)

```bash
# Install Helper Scripts
bash <(curl -s https://raw.githubusercontent.com/tteck/Proxmox/main/install-core.sh)

# Features:
# - Helper scripts menu
# - Container templates
# - OS templates
# - System optimizations
```

### One-Click Install Scripts

#### Home Assistant LXC
```bash
# Install Home Assistant
bash <(curl -s https://raw.githubusercontent.com/tteck/Proxmox/main/install-hassio.sh)
```

#### AdGuard Home
```bash
# Install AdGuard
bash <(curl -s https://raw.githubusercontent.com/tteck/Proxmox/main/install-adguard.sh)
```

#### Pi-hole
```bash
# Install Pi-hole
bash <(curl -s https://raw.githubusercontent.com/tteck/Proxmox/main/install-pihole.sh)
```

#### Docker
```bash
# Install Docker
bash <(curl -s https://raw.githubusercontent.com/tteck/Proxmox/main/install-docker.sh)
```

#### CasaOS
```bash
# Install CasaOS
bash <(curl -s https://raw.githubusercontent.com/tteck/Proxmox/main/install-casaos.sh)
```

#### Portainer
```bash
# Install Portainer
bash <(curl -s https://raw.githubusercontent.com/tteck/Proxmox/main/install-portainer.sh)
```

---

## Helper Scripts

### Auto Start/Stop

```bash
#!/bin/bash
# autostart-manager.sh

# Create autostart configuration
cat > /etc/pve/vm-autostart.conf << EOF
# Format: VMID;START_DELAY;STOP_TIMEOUT
100;30;60
101;30;60
200;15;30
EOF

# Apply
qm start $(awk -F';' '{print $1}' /etc/pve/vm-autostart.conf | grep -v '^#')
```

### Auto Snapshot

```bash
#!/bin/bash
# auto-snapshot.sh

VM_LIST="100 101 102"
SNAPSHOT_DIR="/mnt/pve/snapshots"
RETENTION_DAYS=7

mkdir -p $SNAPSHOT_DIR

for VMID in $VM_LIST; do
    DATE=$(date +%Y%m%d-%H%M)
    echo "Snapshot VM $VMID..."
    qm snapshot $VMID auto-$DATE
done

# Cleanup old snapshots
find $SNAPSHOT_DIR -name "auto-*" -mtime +$RETENTION_DAYS -delete
```

### Resource Monitor

```bash
#!/bin/bash
# resource-monitor.sh

ALERT_THRESHOLD=80

# Check CPU
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
CPU_INT=${CPU%.*}

if [ $CPU_INT -gt $ALERT_THRESHOLD ]; then
    echo "ALERT: CPU at ${CPU}%"
fi

# Check Memory
MEM=$(free | grep Mem)
TOTAL=$(echo $MEM | awk '{print $2}')
USED=$(echo $MEM | awk '{print $3}')
MEM_PCT=$((USED * 100 / TOTAL))

if [ $MEM_PCT -gt $ALERT_THRESHOLD ]; then
    echo "ALERT: Memory at ${MEM_PCT}%"
fi

# Check Storage
for STORE in $(pvesm status | awk 'NR>1 {print $1}'); do
    USED=$(pvesm status $STORE | awk '/used/ {print $3}')
    TOTAL=$(pvesm status $STORE | awk '/total/ {print $2}')
    STORE_PCT=$((USED * 100 / TOTAL))
    
    if [ $STORE_PCT -gt $ALERT_THRESHOLD ]; then
        echo "ALERT: Storage $STORE at ${STORE_PCT}%"
    fi
done
```

### Performance Tuner

```bash
#!/bin/bash
# performance-tuner.sh

# Optimize kernel parameters
cat >> /etc/sysctl.conf << EOF

# Proxmox optimizations
vm.swappiness=10
vm.min_free_kbytes=65536
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 87380 16777216
EOF

# Apply
sysctl -p

# Optimize GRUB
sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="quiet"/GRUB_CMDLINE_LINUX_DEFAULT="quiet transparent_hugepage=always"/' /etc/default/grub
update-grub
```

---

## Home Lab Scripts

### Home Lab Dashboard

```bash
#!/bin/bash
# homelab-status.sh

echo "╔══════════════════════════════════════════╗"
echo "║     Proxmox Home Lab Status              ║"
echo "╚══════════════════════════════════════════╝"
echo ""

echo "📊 NODE INFORMATION"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
hostname
echo "Uptime: $(uptime | cut -d',' -f1 | cut -d' ' -f4)"
echo "Kernel: $(uname -r)"
echo ""

echo "💻 VIRTUAL MACHINES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
for VM in $(qm list | awk 'NR>1 {print $1}' | tr -d '*'); do
    [ -z "$VM" ] && continue
    STATUS=$(qm status $VM | grep -oP 'status: \K\S+')
    NAME=$(qm config $VM | grep -oP 'name: \K\S+')
    MEM=$(qm config $VM | grep -oP 'memory: \K\d+')
    echo "VM $VM ($NAME): $STATUS | ${MEM}MB"
done
echo ""

echo "🖴 CONTAINERS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
for CT in $(pct list | awk 'NR>1 {print $1}'); do
    [ -z "$CT" ] && continue
    STATUS=$(pct status $CT | grep -oP 'status: \K\S+')
    HOST=$(pct config $CT | grep -oP 'hostname: \K\S+')
    MEM=$(pct config $CT | grep -oP 'memory: \K\d+')
    echo "CT $CT ($HOST): $STATUS | ${MEM}MB"
done
echo ""

echo "💾 STORAGE"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
pvesm status | awk 'NR>1 {printf "  %-15s %s / %s (%s)\n", $1, $3, $2, $4}'
echo ""

echo "🌡️ RESOURCES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}')"
free -h | awk '/Mem:/ {print "RAM: " $3 "/" $2}'
df -h / | awk 'NR==2 {print "Disk: " $3 "/" $2}'
```

### Service Monitor

```bash
#!/bin/bash
# service-monitor.sh

check_service() {
    SERVICE=$1
    if systemctl is-active --quiet $SERVICE; then
        echo "✓ $SERVICE"
    else
        echo "✗ $SERVICE - FAILED"
        ((FAILURES++))
    fi
}

echo "Checking services..."
FAILURES=0

check_service "pveproxy"
check_service "pvedaemon"
check_service "pveha-lrm"
check_service "corosync"
check_service "pmxcfs"

if [ $FAILURES -eq 0 ]; then
    echo "All services OK!"
    exit 0
else
    echo "Issues found: $FAILURES"
    exit 1
fi
```

### Network Diagnostics

```bash
#!/bin/bash
# network-diag.sh

echo "=== Network Diagnostics ==="
echo ""

echo "📺 Network Interfaces:"
ip -brief addr show | grep -v "DOWN\|LOOPBACK"
echo ""

echo "🌉 Bridges:"
bridge link show
echo ""

echo "🔗 Bonds:"
for bond in /sys/class/net/*/bonding; do
    [ -e "$bond" ] || continue
    BOND=$(basename $(dirname $bond))
    echo "$BOND:"
    cat $bond/mode 2>/dev/null | head -1
    echo "  Active: $(cat $bond/active_slave 2>/dev/null)"
    echo "  Slaves: $(cat $bond/slaves 2>/dev/null)"
done
echo ""

echo "📊 Traffic:"
ip -s link show | grep -A1 "RX:" | head -3
```

---

## Container Templates

### Common Container Setup

#### Nginx + PHP
```bash
#!/bin/bash
# setup-nginx-php.sh

CTID=$1

pct exec $CTID -- bash -c "
apt update
apt install -y nginx php php-fpm
systemctl enable nginx
systemctl start nginx
"

echo "Nginx + PHP installed!"
```

#### Docker + Portainer
```bash
#!/bin/bash
# setup-docker.sh

CTID=$1

pct exec $CTID -- bash -c "
apt update
apt install -y docker.io docker-compose
usermod -aG docker root
systemctl enable docker
systemctl start docker

# Install Portainer
docker run -d --name portainer -p 9000:9000 --restart always -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
"

echo "Docker + Portainer installed!"
```

#### AdGuard Home
```bash
#!/bin/bash
# setup-adguard.sh

CTID=$1

pct exec $CTID -- bash -c "
apt update
apt install -y curl
curl -sfL https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v

# Enable service
systemctl enable AdGuardHome
systemctl start AdGuardHome
"

echo "AdGuard Home installed!"
```

#### Pi-hole
```bash
#!/bin/bash
# setup-pihole.sh

CTID=$1
PASSWORD=$2

pct exec $CTID -- bash -c "
apt update
apt install -y curl git bash
git clone --depth 1 https://github.com/pi-hole/pi-hole.git /etc/pihole
cd /etc/pihole
 bash -c \"\

if [ -n '$PASSWORD' ]; then
    echo 'webpassword=' | echo -e '\$PASSWORD' | sha256sum | cut -d' ' -f1 > /etc/pihole/.setupVariables
fi
\"
autoet install

systemctl start pihole-FTL
systemctl enable pihole-FTL
"

echo "Pi-hole installed!"
```

---

## VM Optimizations

### VirtIO Driver Installation

```bash
#!/bin/bash
# install-virtio.sh

VMID=$1

# Add VirtIO ISO
qm set $VMID --ide2 local:iso/virtio-win.iso,media=cdrom

# In Windows VM:
# 1. Open Device Manager
# 2. Update drivers from virtio-win.iso
# 3. Select all drivers
```

### GPU Passthrough Script

```bash
#!/bin/bash
# gpu-passthrough.sh

GPU_PCI=$1
VMID=$2

if [ -z "$GPU_PCI" ] || [ -z "$VMID" ]; then
    echo "Usage: $0 <gpu_pci> <vmid>"
    echo "Example: $0 01:00 100"
    exit 1
fi

echo "Adding GPU $GPU_PCI to VM $VMID..."

qm set $VMID --hostpci0 ${GPU_PCI},pcie=1,x-vga=1

echo "GPU passthrough enabled!"
```

### Performance Fixes

```bash
#!/bin/bash
# vm-fix-perf.sh

VMID=$1

# Enable host CPU
qm set $VMID --cpu host

# Enable VirtIO
qm set $VMID --scsi0 local:,iothread=1

# Enable multi-queue
qm set $VMID --net0 virtio,bridge=vmbr0,queues=4

# Enable balloon
qm set $VMID --balloon 2048

echo "Performance optimizations applied!"
```

---

## Networking Scripts

### VLAN Setup

```bash
#!/bin/bash
# setup-vlan.sh

VLAN_ID=$1
BRIDGE=$2
IP=$3
NETMASK=$4

if [ -z "$VLAN_ID" ] || [ -z "$BRIDGE" ] || [ -z "$IP" ]; then
    echo "Usage: $0 <vlan_id> <bridge> <ip> [netmask]"
    exit 1
fi

NETMASK=${NETMASK:-255.255.255.0}

# Create VLAN interface
cat >> /etc/network/interfaces.d/vlan$VLAN_ID << EOF

auto ${BRIDGE}.${VLAN_ID}
iface ${BRIDGE}.${VLAN_ID} inet static
    address $IP
    netmask $NETMASK
    vlan-raw-device ${BRIDGE}
EOF

systemctl networking restart

echo "VLAN $VLAN_ID configured on $BRIDGE with IP $IP"
```

### Create Internal Network

```bash
#!/bin/bash
# create-internal-net.sh

NETWORK_NAME=$1
NETWORK_CIDR=$2

if [ -z "$NETWORK_NAME" ] || [ -z "$NETWORK_CIDR" ]; then
    echo "Usage: $0 <network_name> <cidr>"
    echo "Example: $0 vmbr10 10.10.10.0/24"
    exit 1
fi

# Create bridge
cat >> /etc/network/interfaces.d/$NETWORK_NAME << EOF

auto $NETWORK_NAME
iface $NETWORK_NAME inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
EOF

systemctl networking restart

echo "Internal network $NETWORK_NAME ($NETWORK_CIDR) created!"
```

---

## Storage Scripts

### LVM Extension

```bash
#!/bin/bash
# extend-lvm.sh

LV_NAME=$1
SIZE_GB=$2

if [ -z "$LV_NAME" ] || [ -z "$SIZE_GB" ]; then
    echo "Usage: $0 <lv_name> <size_gb>"
    echo "Example: $0 data 100"
    exit 1
fi

echo "Extending LV $LV_NAME by ${SIZE_GB}GB..."

lvextend -L +${SIZE_GB}G /dev/pve/$LV_NAME

echo "LV extended!"
```

### ZFS Pool Create

```bash
#!/bin/bash
# create-zfs-pool.sh

POOL_NAME=$1
DISKS=$2

if [ -z "$POOL_NAME" ] || [ -z "$DISKS" ]; then
    echo "Usage: $0 <pool_name> <disks>"
    echo "Example: $0 tank sda sdb"
    echo "Raid: create-zfs-pool.sh tank sda sdb sdc (for raidz)"
    exit 1
fi

echo "Creating ZFS pool $POOL_NAME..."

case $# in
    2)
        zpool create -f $POOL_NAME mirror $2 $3
        ;;
    3)
        zpool create -f $POOL_NAME mirror $2 $3
        ;;
    *)
        DISK_LIST=$(echo $* | cut -d' ' -f2-)
        zpool create -f -o ashift=12 $POOL_NAME raidz $DISK_LIST
        ;;
esac

# Add to Proxmox
pvesm add zfs $POOL_NAME --pool $POOL_NAME --content rootdir,images

echo "ZFS pool $POOL_NAME created and added to Proxmox!"
```

---

## Backup Solutions

### rsync Backup

```bash
#!/bin/bash
# rsync-backup.sh

BACKUP_SOURCE="/mnt/pve/backup"
REMOTE_HOST="backup-server"
REMOTE_PATH="/backup/proxmox"
KEY_FILE="/root/.ssh/backup_key"

echo "Syncing backups to $REMOTE_HOST..."

rsync -avz --delete \
    -e "ssh -i $KEY_FILE" \
    $BACKUP_SOURCE/ \
    user@$REMOTE_HOST:$REMOTE_PATH/

echo "Sync complete!"
```

### rclone Backup

```bash
#!/bin/bash
# rclone-backup.sh

SOURCE="/mnt/pve/backup"
REMOTE="gdrive:ProxmoxBackup"

echo "Uploading backups to Google Drive..."

rclone sync $SOURCE $REMOTE -v --progress

echo "Upload complete!"
```

---

## Monitoring Solutions

### Grafana Dashboard

```bash
#!/bin/bash
# setup-grafana.sh

# Install InfluxDB
apt install -y influxdb

# Install Grafana
wget -qO- https://packages.grafana.com/gpg.key | apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" > /etc/apt/sources.list.d/grafana.list
apt update
apt install -y grafana

# Configure
systemctl enable influxdb
systemctl enable grafana-server
systemctl start influxdb
systemctl start grafana-server

echo "Grafana installed at http://$(hostname -I):3000"
```

### Prometheus Exporter

```bash
#!/bin/bash
# setup-prometheus.sh

# Download node_exporter
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz

# Extract
tar xzf node_exporter-*.linux-amd64.tar.gz
cp node_exporter-*.linux-amd64/node_exporter /usr/local/bin/

# Create service
cat > /etc/systemd/system/node_exporter.service << EOF
[Unit]
Description=Prometheus Node Exporter

[Service]
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable node_exporter
systemctl start node_exporter

echo "Node exporter started on port 9100"
```

---

## Keywords

#community #scripts #automation #homelab #monitoring #backup #optimization #templates #helper #tools

## Links

- [[Proxmox-VE]] - Main documentation
- [[Scripts]] - More scripts
- [[VM]] - VM management
- [[Containers]] - Container management
- [[HomeLab]] - Home lab setup
---

[[index|Back to Proxmox VE]]
