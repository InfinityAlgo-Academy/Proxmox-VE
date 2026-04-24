# Container Automation Scripts

## Overview

Automation scripts for LXC containers. Create, manage, and maintain containers automatically.

## Container Creation

### Basic Create Script

```bash
#!/bin/bash
# create-ct.sh - Create container

VMID=$1
TEMPLATE=${2:-debian-12}
MEMORY=${3:-2048}
CORES=${4:-2}
IP=${5:-dhcp}

# Create container
pct create $VMID local:vztmpl/$TEMPLATE.tar.gz \
  --rootfs local:8 \
  --memory $MEMORY \
  --cores $CORES \
  --net0 name=eth0,bridge=vmbr0,ip=$IP \
  --hostname ct-$VMID

# Enable autostart
pct set $VMID --onboot 1

# Start
pct start $VMID

echo "Container $VMID created"
```

### Full Setup Script

```bash
#!/bin/bash
# setup-ct.sh - Complete container setup

set -euo pipefail

VMID=$1
HOSTNAME=$2
TEMPLATE=${3:-debian-12}
PACKAGES=${4:-curl,wget,git}

# Create container
pct create $VMID local:vztmpl/$TEMPLATE.tar.gz \
  --rootfs local:10 \
  --memory 4096 \
  --cores 4 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --hostname $HOSTNAME

pct set $VMID --onboot 1 --startup-delay 10

# Start
pct start $VMID

# Wait for network
sleep 10

# Update
pct exec $VMID -- apt update

# Install packages
pct exec $VMID -- apt install -y $PACKAGES

echo "Container $VMID ready"
```

## Management Scripts

### Start/Stop All

```bash
#!/bin/bash
# manage-ct.sh - Manage containers

ACTION=$1
shift

case $ACTION in
    start)
        for VMID in "$@"; do
            pct start $VMID
            echo "Started CT $VMID"
        done
        ;;
    stop)
        for VMID in "$@"; do
            pct stop $VMID
            echo "Stopped CT $VMID"
        done
        ;;
    restart)
        for VMID in "$@"; do
            pct restart $VMID
            echo "Restarted CT $VMID"
        done
        ;;
esac
```

### Execute in All

```bash
#!/bin/bash
# exec-all.sh - Execute in all containers

COMMAND="$@"

for VMID in $(pct list | grep running | awk '{print $1}'); do
    echo "=== CT $VMID ==="
    pct exec $VMID -- bash -c "$COMMAND"
done
```

## Update Scripts

### Update All Containers

```bash
#!/bin/bash
# update-ct.sh - Update all containers

for VMID in $(pct list | grep running | awk '{print $1}'); do
    echo "Updating CT $VMID..."
    pct exec $VMID -- bash -c "apt update && apt upgrade -y"
    echo "CT $VMID updated"
done

echo "All updates complete"
```

## Backup Containers

### Container Backup

```bash
#!/bin/bash
# backup-ct.sh - Backup container

VMID=$1
STORAGE=${2:-local}

pct stop $VMID
vzdump --mode stop --vmid $VMID --storage $STORAGE
pct start $VMID

echo "Container $VMID backed up"
```

## Keywords

#container #lxc #automation #scripting #management

## Related Articles

- [[Automation]]
- [[Containers]]
- [[Scripts]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
