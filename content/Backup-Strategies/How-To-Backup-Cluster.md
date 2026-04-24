# How to Back Up Cluster - Complete Guide

## Question: How do I back up the entire Proxmox cluster?

### Answer: Back up each component separately

---

## What to Back Up

### Components
1. **Configuration**: /etc/pve/
2. **VMs**: Individual backups
3. **Templates**: /var/lib/vz/template/
4. **Certificates**: /etc/pve/pve-*.pem

---

## How to Back Up Configuration

### Script
```bash
#!/bin/bash
# backup-cluster-config.sh

BACKUP_DIR="/mnt/backup/cluster"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR/$DATE

# Backup /etc/pve (cluster configuration)
tar -czf $BACKUP_DIR/$DATE/pve-config.tar.gz /etc/pve/

# Backup certificates
tar -czf $BACKUP_DIR/$DATE/certs.tar.gz /etc/pve/pve-*.pem

# Backup user configuration
tar -czf $BACKUP_DIR/$DATE/users.tar.gz /etc/pve/user.cfg

echo "Cluster config backed up to $BACKUP_DIR/$DATE"
```

---

## How to Restore Cluster Config

### Step 1: Reinstall Proxmox
Follow standard installation

### Step 2: Restore Config
```bash
# Extract configuration
tar -xzf /mnt/backup/pve-config.tar.gz -C /

# Rebuild cluster
pvecm create cluster-name
```

---

## How to Back Up All VMs

### Full VM Backup Script
```bash
#!/bin/bash
# backup-all-vms.sh

STORAGE=local

# Get all VMs
VM_LIST=$(qm list | grep running | awk '{print $1}')
CT_LIST=$(pct list | grep running | awk '{print $1}')

# Backup VMs
for VMID in $VM_LIST; do
    echo "Backing up VM $VMID"
    vzdump --mode suspend --vmid $VMID --storage $STORAGE --compress zstd
done

# Backup Containers
for CTID in $CT_LIST; do
    echo "Backing up Container $CTID"
    vzdump --mode suspend --vmid $CTID --storage $STORAGE --compress zstd
done
```

---

## Keywords

#cluster-backup #configuration #how-to #disaster-recovery

## Related Articles

- [[Backup]]
- [[Cluster]]
- [[Disaster-Recovery]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
