# Proxmox VE Scripts & Automation

## Table of Contents

1. [Script Basics](#script-basics)
2. [VM Management Scripts](#vm-management-scripts)
3. [Container Scripts](#container-scripts)
4. [Backup Scripts](#backup-scripts)
5. [Monitoring Scripts](#monitoring-scripts)
6. [Storage Scripts](#storage-scripts)
7. [Network Scripts](#network-scripts)
8. [Maintenance Scripts](#maintenance-scripts)
9. [Automation with Cron](#automation-with-cron)
10. [Advanced Scripts](#advanced-scripts)

---

## Script Basics

### VM ID Functions

```bash
#!/bin/bash
# vm-functions.sh

# Get next available VM ID
get_next_vmid() {
    pvesh get /cluster/nextid
}

# Check if VM exists
vm_exists() {
    local vmid=$1
    qm status $vmid >/dev/null 2>&1 && echo "yes" || echo "no"
}

# Get VM status
get_vm_status() {
    local vmid=$1
    qm status $vmid 2>/dev/null | grep -oP 'status: \K\S+'
}

# Get VM name
get_vm_name() {
    local vmid=$1
    qm config $vmid 2>/dev/null | grep -oP 'name: \K\S+'
}

# Get VM Notes
get_vm_notes() {
    local vmid=$1
    qm config $vmid 2>/dev/null | grep -oP 'description: \K\S+'
}
```

### Container Functions

```bash
#!/bin/bash
# lxc-functions.sh

# Get next available CT ID
get_next_ctid() {
    pvesh get /cluster/nextid
}

# Check if CT exists
ct_exists() {
    local ctid=$1
    pct status $ctid >/dev/null 2>&1
}

# Get CT status
get_ct_status() {
    local ctid=$1
    pct status $ctid 2>/dev/null | grep -oP 'status: \K\S+'
}
```

---

## VM Management Scripts

### Clone VM Script

```bash
#!/bin/bash
# clone-vm.sh

# Variables
SOURCE_VM=$1
TARGET_VM=$2
TARGET_NAME=$3
STORAGE=${4:-local}

if [ -z "$SOURCE_VM" ] || [ -z "$TARGET_VM" ] || [ -z "$TARGET_NAME" ]; then
    echo "Usage: $0 <source_vm> <target_vm> <target_name> [storage]"
    exit 1
fi

echo "Cloning VM $SOURCE_VM to $TARGET_VM..."

# Clone VM
qm clone $SOURCE_VM $TARGET_VM --full --name "$TARGET_NAME" --storage $STORAGE

if [ $? -eq 0 ]; then
    echo "VM cloned successfully!"
    echo "Starting VM $TARGET_VM..."
    qm start $TARGET_VM
else
    echo "Failed to clone VM"
    exit 1
fi
```

### Multiple VM Clone

```bash
#!/bin/bash
# clone-multiple-vms.sh

BASE_VM=$1
START_ID=$2
COUNT=$3
NAME_PREFIX=${4:-vm}
STORAGE=${5:-local}

if [ -z "$BASE_VM" ] || [ -z "$START_ID" ] || [ -z "$COUNT" ]; then
    echo "Usage: $0 <base_vm> <start_id> <count> [name_prefix] [storage]"
    exit 1
fi

for i in $(seq 1 $COUNT); do
    VMID=$((START_ID + i - 1))
    VMNAME="${NAME_PREFIX}-${i}"
    
    echo "Cloning $BASE_VM -> $VMID ($VMNAME)..."
    qm clone $BASE_VM $VMID --full --name "$VMNAME" --storage $STORAGE
    
    if [ $? -eq 0 ]; then
        echo "VM $VMID created successfully"
    else
        echo "Failed to create VM $VMID"
    fi
done

echo "All VMs cloned!"
```

### Batch Start/Stop VMs

```bash
#!/bin/bash
# vm-batch.sh

ACTION=$1
shift
VMIDS="$@"

if [ -z "$ACTION" ] || [ -z "$VMIDS" ]; then
    echo "Usage: $0 <start|stop|restart> <vm_id> [vm_id ...]"
    exit 1
fi

case $ACTION in
    start)
        for vmid in $VMIDS; do
            echo "Starting VM $vmid..."
            qm start $vmid
        done
        ;;
    stop)
        for vmid in $VMIDS; do
            echo "Stopping VM $vmid..."
            qm stop $vmid
        done
        ;;
    restart)
        for vmid in $VMIDS; do
            echo "Restarting VM $vmid..."
            qm stop $vmid && qm start $vmid
        done
        ;;
    *)
        echo "Unknown action: $ACTION"
        exit 1
        ;;
esac

echo "Done!"
```

### Create VM from ISO

```bash
#!/bin/bash
# create-vm.sh

NAME=$1
VMID=$2
ISO=$3
STORAGE=${4:-local}
CORES=${5:-2}
MEMORY=${6:-4096}
DISK=${7:-32G}

if [ -z "$NAME" ] || [ -z "$VMID" ] || [ -z "$ISO" ]; then
    echo "Usage: $0 <name> <vmid> <iso> [storage] [cores] [memory] [disk_size]"
    echo ""
    echo "Available ISOs:"
    ls /var/lib/vz/template/iso/
    exit 1
fi

echo "Creating VM $NAME (ID: $VMID)..."

# Create VM
qm create $VMID \
    --name "$NAME" \
    --memory $MEMORY \
    --cores $CORES \
    --scsi0 ${STORAGE}:vm-${VMID}-disk-0,size=${DISK} \
    --net0 virtio,bridge=vmbr0 \
    --cdrom local:iso/${ISO} \
    --boot order=ide2;scsi0 \
    --onboot 1

if [ $? -eq 0 ]; then
    echo "VM created successfully!"
    echo "Starting VM $VMID..."
    qm start $VMID
else
    echo "Failed to create VM"
    exit 1
fi
```

### Delete VM

```bash
#!/bin/bash
# delete-vm.sh

VMID=$1
BACKUP=${2:-no}

if [ -z "$VMID" ]; then
    echo "Usage: $0 <vmid> [backup]"
    echo "backup: yes/no (default: no)"
    exit 1
fi

# Check VM exists
if ! qm status $VMID >/dev/null 2>&1; then
    echo "VM $VMID does not exist"
    exit 1
fi

# Stop VM
echo "Stopping VM $VMID..."
qm stop $VMID

# Optional backup
if [ "$BACKUP" = "yes" ]; then
    echo "Backing up VM $VMID..."
    vzdump $VMID --mode stop --storage local-backup --compress lzo
fi

# Delete VM
echo "Deleting VM $VMID..."
qm destroy $VMID

echo "VM $VMID deleted!"
```

---

## Container Scripts

### Create Container

```bash
#!/bin/bash
# create-ct.sh

HOSTNAME=$1
CTID=$2
TEMPLATE=$3
ROOTFS=${4:-8}
RAM=${5:-1024}
CORES=${6:-1}
NETWORK=${7:-dhcp}

if [ -z "$HOSTNAME" ] || [ -z "$CTID" ] || [ -z "$TEMPLATE" ]; then
    echo "Usage: $0 <hostname> <ctid> <template> [rootfs_gb] [ram_mb] [cores] [dhcp|ip]"
    echo ""
    echo "Available templates:"
    pveam list standard
    exit 1
fi

echo "Creating container $HOSTNAME (ID: $CTID)..."

# Create container
pct create $CTID local:vztmpl/${TEMPLATE}.tar.gz \
    --rootfs local:$ROOTFS \
    --memory $RAM \
    --cores $CORES \
    --net0 name=eth0,bridge=vmbr0,ip=${NETWORK},type=veth \
    --hostname $HOSTNAME \
    --password rootpassword \
    --unprivileged 1

if [ $? -eq 0 ]; then
    echo "Container created!"
    echo "Starting $CTID..."
    pct start $CTID
else
    echo "Failed to create container"
    exit 1
fi
```

### Clone Container

```bash
#!/bin/bash
# clone-ct.sh

SOURCE_CT=$1
TARGET_CT=$2
TARGET_HOSTNAME=$3

if [ -z "$SOURCE_CT" ] || [ -z "$TARGET_CT" ] || [ -z "$TARGET_HOSTNAME" ]; then
    echo "Usage: $0 <source_ct> <target_ct> <target_hostname>"
    exit 1
fi

echo "Cloning CT $SOURCE_CT to $TARGET_CT..."

pct stop $SOURCE_CT
pct snapshot $SOURCE_CT base

pct clone $SOURCE_CT $TARGET_CT --full --hostname $TARGET_HOSTNAME

if [ $? -eq 0 ]; then
    echo "Container cloned!"
    pct start $TARGET_CT
else
    echo "Failed to clone"
    exit 1
fi
```

### Batch Container Operations

```bash
#!/bin/bash
# ct-batch.sh

ACTION=$1
shift
CTIDS="$@"

case $ACTION in
    start)
        for ctid in $CTIDS; do
            echo "Starting CT $ctid..."
            pct start $ctid
        done
        ;;
    stop)
        for ctid in $CTIDS; do
            echo "Stopping CT $ctid..."
            pct stop $ctid --timeout 60
        done
        ;;
    restart)
        for ctid in $CTIDS; do
            echo "Restarting CT $ctid..."
            pct restart $ctid
        done
        ;;
    *)
        echo "Usage: $0 <start|stop|restart> <ct_id>..."
        exit 1
        ;;
esac

echo "Done!"
```

### Update Container

```bash
#!/bin/bash
# update-ct.sh

CTID=$1

if [ -z "$CTID" ]; then
    echo "Usage: $0 <ctid>"
    exit 1
fi

echo "Updating CT $CTID..."

# Create snapshot
pct snapshot $CTID pre-update

# Stop container
pct stop $CTID

# Get container IP
CT_IP=$(pct exec $CTID -- hostname -I | awk '{print $1}')

echo "Container IP: $CT_IP"

# Update
pct start $CTID

sleep 5

# Update packages
pct exec $CTID -- bash -c "apt update && apt upgrade -y"

echo "Update complete!"
```

---

## Backup Scripts

### Automated Backup

```bash
#!/bin/bash
# backup-all.sh

# Configuration
STORAGE=${1:-local}
COMPRESS=${2:-lzo}
MODE=${3:-snapshot}
EXCLUDES="--exclude .Run"

# Get all VMs and CTs
VMS=$(qm list | awk 'NR>1 {print $1}')
CTS=$(pct list | awk 'NR>1 {print $1}')

echo "Backing up all VMs and containers..."

# Backup VMs
for vmid in $VMS; do
    vmid=$(echo $vmid | tr -d '*')
    [ -z "$vmid" ] && continue
    echo "Backing up VM $vmid..."
    vzdump $vmid --$MODE --storage $STORAGE --$COMPRESS $EXCLUDES
done

# Backup CTs
for ctid in $CTS; do
    [ -z "$ctid" ] && continue
    echo "Backing up CT $ctid..."
    vzdump $ctid --$MODE --storage $STORAGE --$COMPRESS $EXCLUDES
done

echo "All backups complete!"
echo "Backups location: /mnt/$STORAGE/dump/"
```

### Selective Backup

```bash
#!/bin/bash
# backup-select.sh

# Backup specific VMs/CTs
VM_LIST="100 101 102"
CT_LIST="200 201"
STORAGE="local"

echo "Backing up selected VMs and CTs..."

for vmid in $VM_LIST; do
    echo "Backing up VM $vmid..."
    vzdump $vmid --mode snapshot --storage $STORAGE
done

for ctid in $CT_LIST; do
    echo "Backing up CT $ctid..."
    vzdump $ctid --mode snapshot --storage $STORAGE
done

echo "Done!"
```

### Clean Old Backups

```bash
#!/bin/bash
# clean-backups.sh

STORAGE=$1
DAYS=${2:-7}

if [ -z "$STORAGE" ]; then
    echo "Usage: $0 <storage> [days]"
    exit 1
fi

echo "Cleaning backups older than $DAYS days from $STORAGE..."

BACKUP_DIR=$(pvesm path $STORAGE)
FIND_CMD="find $BACKUP_DIR/dump -type f -name '*.tar.gz' -mtime +$DAYS -delete"

eval $FIND_CMD

echo "Old backups cleaned!"
echo "Remaining backups:"
ls -la $BACKUP_DIR/dump/
```

### Backup to Remote

```bash
#!/bin/bash
# backup-remote.sh

REMOTE_HOST="backup-server"
REMOTE_PATH="/mnt/backup/proxmox"
STORAGE="local"

echo "Backing up to remote server $REMOTE_HOST..."

# Get all VMs and CTs
VMS=$(qm list | awk 'NR>1 {print $1}')
CTS=$(pct list | awk 'NR>1 {print $1}')

DATE=$(date +%Y%m%d)

# Backup and sync
for vmid in $VMS; do
    vmid=$(echo $vmid | tr -d '*')
    [ -z "$vmid" ] && continue
    echo "Backing up VM $vmid..."
    vzdump $vmid --mode stop --storage $STORAGE
    rsync -avz $BACKUP_DIR/dump/vzdump-qemu-$vmid-$DATE* user@$REMOTE_HOST:$REMOTE_PATH/
done

echo "Remote backup complete!"
```

### Restore Script

```bash
#!/bin/bash
# restore.sh

BACKUP_FILE=$1
TARGET_ID=$2

if [ -z "$BACKUP_FILE" ] || [ -z "$TARGET_ID" ]; then
    echo "Usage: $0 <backup_file> <target_id>"
    echo ""
    echo "Available backups:"
    ls -la /mnt/local/dump/
    exit 1
fi

echo "Restoring $BACKUP_FILE to ID $TARGET_ID..."

# Detect type and restore
if [[ $BACKUP_FILE == *"qemu"* ]]; then
    qmrestore /mnt/local/dump/$BACKUP_FILE $TARGET_ID
elif [[ $BACKUP_FILE == *"lxc"* ]]; then
    pct restore /mnt/local/dump/$BACKUP_FILE $TARGET_ID
fi

echo "Restore complete!"
```

---

## Monitoring Scripts

### Resource Check

```bash
#!/bin/bash
# resource-check.sh

THRESHOLD_CPU=${1:-80}
THRESHOLD_MEM=${2:-80}
THRESHOLD_DISK=${3:-80}

echo "=== Proxmox Resource Check ==="
echo "Threshold - CPU: $THRESHOLD_CPU%, Memory: $THRESHOLD_MEM%, Disk: $THRESHOLD_DISK%"
echo ""

# Node resources
echo "=== Node Resources ==="
echo "CPU Load:"
uptime | awk -F'load average:' '{print "  " $NF}'

echo ""
echo "Memory:"
free -h | awk '/Mem:/ {print "  Used: " $3 ", Free: " $4 ", Total: " $2}'

echo ""
echo "Disk:"
df -h / | awk 'NR==2 {print "  Used: " $3 " (" $5 "), Free: " $4}'

echo ""
# Check VMs
echo "=== VM Status ==="
for vm in $(qm list | awk 'NR>1 {print $1}' | tr -d '*'); do
    [ -z "$vm" ] && continue
    status=$(qm status $vm 2>/dev/null | grep -oP 'status: \K\S+' || echo "unknown")
    echo "VM $vm: $status"
done

echo ""
# Check CTs
echo "=== Container Status ==="
for ct in $(pct list | awk 'NR>1 {print $1}'); do
    [ -z "$ct" ] && continue
    status=$(pct status $ct 2>/dev/null | grep -oP 'status: \K\S+' || echo "unknown")
    echo "CT $ct: $status"
done
```

### Health Check

```bash
#!/bin/bash
# health-check.sh

ALERT_EMAIL="admin@example.com"

echo "=== Proxmox Health Check ==="

ERRORS=0

# Check node
echo -n "Node status: "
if uptime | grep -q "day"; then
    echo "OK ($(uptime | cut -d',' -f1 | cut -d' ' -f4))"
else
    echo "WARNING - Uptime low"
    ((ERRORS++))
fi

# Check services
echo -n "Services: "
SERVICES_OK=1
for svc in pveproxy pvedaemon pveha-lrm; do
    if ! systemctl is-active $svc >/dev/null 2>&1; then
        echo "FAILED"
        echo "  - $svc is not running"
        SERVICES_OK=0
        ((ERRORS++))
    fi
done
[ $SERVICES_OK -eq 1 ] && echo "OK"

# Check storage
echo -n "Storage: "
for store in $(pvesm status | awk 'NR>1 {print $1}'); do
    status=$(pvesm status $store | grep -oP 'active: \K\S+' || echo "unknown")
    if [ "$status" = "1" ]; then
        echo -n "OK "
    else
        echo "FAILED ($store)"
        ((ERRORS++))
    fi
done
echo ""

# Check VMs/CTs
echo -n "VMs: "
NOT_RUNNING=0
for vm in $(qm list | awk 'NR>1 {print $1}' | tr -d '*'); do
    status=$(qm status $vm 2>/dev/null | grep -oP 'status: \K\S+' || echo "unknown")
    if [ "$status" != "running" ]; then
        echo "VM $vm: $status"
        ((NOT_RUNNING++))
    fi
done
[ $NOT_RUNNING -eq 0 ] && echo "All running"

# Summary
echo ""
if [ $ERRORS -gt 0 ]; then
    echo "ISSUES FOUND: $ERRORS"
    echo "Check system immediately!"
    exit 1
else
    echo "All systems OK"
    exit 0
fi
```

### Daily Report

```bash
#!/bin/bash
# daily-report.sh

REPORT_FILE="/root/reports/daily-$(date +%Y%m%d).txt"
mkdir -p $(dirname $REPORT_FILE)

{
    echo "=== Proxmox Daily Report - $(date) ==="
    echo ""
    echo "=== Node Info ==="
    hostname
    uptime
    echo ""
    echo "=== Resource Usage ==="
    free -h
    df -h
    echo ""
    echo "=== VM Status ==="
    qm list
    echo ""
    echo "=== Container Status ==="
    pct list
    echo ""
    echo "=== Storage Status ==="
    pvesm status
    echo ""
    echo "=== Recent Backups ==="
    ls -lart /mnt/local/dump/ | tail -10
} > $REPORT_FILE

echo "Report saved to $REPORT_FILE"
```

---

## Storage Scripts

### Check Storage Space

```bash
#!/bin/bash
# check-storage.sh

THRESHOLD=${1:-90}

echo "=== Storage Check (Threshold: $THRESHOLD%) ==="

for store in $(pvesm status | awk 'NR>1 {print $1}'); do
    used=$(pvesm status $store | awk 'NR>2 {print $3}' | tail -1)
    total=$(pvesm status $store | awk 'NR>2 {print $2}' | tail -1)
    percentage=$(echo "scale=0; $used*100/$total" | bc)
    
    if [ $percentage -gt $THRESHOLD ]; then
        echo "WARNING: $store is at ${percentage}% ($used / $total)"
    else
        echo "OK: $store is at ${percentage}%"
    fi
done
```

### Add Storage

```bash
#!/bin/bash
# add-storage.sh

TYPE=$1
NAME=$2
PATH=$3
CONTENT=${4:-rootdir,images}

if [ -z "$TYPE" ] || [ -z "$NAME" ] || [ -z "$PATH" ]; then
    echo "Usage: $0 <type> <name> <path> [content]"
    echo "Types: dir, lvmthin, zfs, nfs, iscsi"
    exit 1
fi

mkdir -p $PATH
pvesm add $TYPE $NAME --path $PATH --content $CONTENT

echo "Storage $NAME added!"
```

### ZFS Snapshot

```bash
#!/bin/bash
# zfs-snapshot.sh

POOL=$1
SNAPSHOT_NAME=${2:-$(date +%Y%m%d)}

if [ -z "$POOL" ]; then
    echo "Usage: $0 <pool> [snapshot_name]"
    exit 1
fi

echo "Creating ZFS snapshot on $POOL..."

zfs snapshot ${POOL}@${SNAPSHOT_NAME}

echo "Snapshot ${SNAPSHOT_NAME} created!"
zfs list -t snapshot | grep $SNAPSHOT_NAME
```

---

## Network Scripts

### Network Restart

```bash
#!/bin/bash
# restart-network.sh

echo "Restarting networking..."

systemctl networking restart

echo "Network restarted!"
ip addr show vmbr0
```

### Port Forwarding

```bash
#!/bin/bash
# port-forward.sh

HOST_IP=$1
HOST_PORT=$2
GUEST_IP=$3
GUEST_PORT=$4

if [ -z "$HOST_IP" ] || [ -z "$HOST_PORT" ] || [ -z "$GUEST_IP" ] || [ -z "$GUEST_PORT" ]; then
    echo "Usage: $0 <host_ip> <host_port> <guest_ip> <guest_port>"
    exit 1
fi

echo "Setting up port forwarding: $HOST_IP:$HOST_PORT -> $GUEST_IP:$GUEST_PORT"

iptables -t nat -A PREROUTING -p tcp -d $HOST_IP --dport $HOST_PORT -j DNAT --to $GUEST_IP:$GUEST_PORT
iptables -A FORWARD -p tcp -d $GUEST_IP --dport $GUEST_PORT -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

# Save
iptables-save > /etc/iptables.rules

echo "Port forwarding configured!"
```

---

## Maintenance Scripts

### Update All VMs

```bash
#!/bin/bash
# update-all.sh

echo "Updating all VMs..."

VMS=$(qm list | awk 'NR>1 {print $1}' | tr -d '*')

for vm in $VMS; do
    [ -z "$vm" ] && continue
    echo "Updating VM $vm..."
    qm stop $vm
    # Add update commands here
    qm start $vm
done

echo "All VMs updated!"
```

### Update Containers

```bash
#!/bin/bash
# update-ct-all.sh

CTS=$(pct list | awk 'NR>1 {print $1}')

echo "Updating all containers..."

for ct in $CTS; do
    [ -z "$ct" ] && continue
    echo "Updating CT $ct..."
    pct exec $ct -- bash -c "apt update && apt upgrade -y"
done

echo "All containers updated!"
```

### System Cleanup

```bash
#!/bin/bash
# cleanup.sh

echo "Proxmox cleanup..."

# Clean logs
find /var/log -name "*.gz" -mtime +30 -delete
find /var/log -name "*.1" -mtime +30 -delete

# Clean old backups
find /mnt/local/dump -name "*.tar.gz" -mtime +30 -delete

# Clean package cache
apt clean

# Clean thumbnails
rm -rf /var/tmp/pve/*

echo "Cleanup complete!"
```

---

## Automation with Cron

### Cron Setup

```bash
# Setup cron jobs

# Edit crontab
crontab -e

# Daily backup at 2 AM
0 2 * * * /root/scripts/backup-all.sh

# Hourly health check
0 * * * * /root/scripts/health-check.sh

# Daily resource report
0 9 * * * /root/scripts/daily-report.sh

# Weekly cleanup
0 3 * * 0 /root/scripts/cleanup.sh

# Save crontab
crontab -l > /root/scripts/crontab
```

---

## Advanced Scripts

### VM Template Creator

```bash
#!/bin/bash
# create-template.sh

VMID=$1
NAME=$2
DESCRIPTION=$3

if [ -z "$VMID" ] || [ -z "$NAME" ]; then
    echo "Usage: $0 <vmid> <name> [description]"
    exit 1
fi

echo "Creating VM template: $NAME"

# Stop VM
qm stop $VMID

# Convert to template
qm template $VMID

# Set description
if [ -n "$DESCRIPTION" ]; then
    qm set $VMID --description "$DESCRIPTION"
fi

echo "Template $NAME created!"
```

### Migration Script

```bash
#!/bin/bash
# migrate-vm.sh

VMID=$1
TARGET_NODE=$2

if [ -z "$VMID" ] || [ -z "$TARGET_NODE" ]; then
    echo "Usage: $0 <vmid> <target_node>"
    exit 1
fi

echo "Migrating VM $VMID to $TARGET_NODE..."

# Check target has space
TARGET_FREE=$(ssh $TARGET_NODE 'df -h / | tail -1 | awk "{print \$4}"')

echo "Target free space: $TARGET_FREE"

# Migrate
qm migrate $VMID $TARGET_NODE --online

echo "Migration complete!"
```

### API Integration

```bash
#!/bin/bash
# api-example.sh

NODE="pve"
USER="root@pam"
TOKEN="YOUR-API-TOKEN"

# Get cluster status
curl -s -H "Authorization: PVEAPIToken=${USER}=${TOKEN}" \
    https://${NODE}:8006/api2/json/nodes

# Get VM list
curl -s -H "Authorization: PVEAPIToken=${USER}=${TOKEN}" \
    https://${NODE}:8006/api2/json/cluster/resources

# Start VM
curl -s -X POST -H "Authorization: PVEAPIToken=${USER}=${TOKEN}" \
    https://${NODE}:8006/api2/json/nodes/pve/qemu/100/status/start
```

---

## Keywords

#scripts #automation #bash #cron #vm #lxc #backup #monitoring #api #migration #template

## Links

- [[Proxmox-VE]] - Main documentation
- [[VM]] - VM management
- [[Containers]] - Container management
- [[Storage]] - Storage management
- [[Backup]] - Backup configuration
- [[Troubleshooting]] - Script debugging
---

[[index|Back to Proxmox VE]]
