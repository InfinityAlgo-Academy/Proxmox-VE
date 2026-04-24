---
title: How to Choose Backup Storage - Complete Guide
---

# How to Choose Backup Storage - Complete Guide

## Question: Where should I store my backups?

### Answer: Multiple storage options available

---

## Option 1: Local Storage (Easiest)

### How to Configure

```bash
# Check current storage
pvesm status

# Local storage is default at /var/lib/vz/dump/
df -h /var/lib/vz
```

### Pros:
- No additional setup
- Fast backup speed
- Already configured

### Cons:
- Same server = same disk failure risk

---

## Option 2: NFS Storage

### How to Configure

**Step 1: Set up NFS server**
```bash
# Install NFS server
apt install nfs-kernel-server

# Create backup directory
mkdir -p /backup
echo "/backup 192.168.1.0/24(rw,sync,no_subtree_check)" >> /etc/exports

# Export
exportfs -a
```

**Step 2: Add NFS in Proxmox**
1. **Datacenter** → **Storage** → **Add** → **NFS**
2. Fill in:
   - **Server**: 192.168.1.200
   - **Export**: /backup
   - **ID**: backup-nfs

**Step 3: CLI Method**
```bash
pvesm add nfs backup-nfs \
  --server 192.168.1.200 \
  --export /backup \
  --content vztmpl,iso,backup
```

### Pros:
- Network accessible
- Can use different server
- Shared between nodes

---

## Option 3: SMB/CIFS

### How to Configure

```bash
# Install CIFS
apt install cifs-utils

# Add SMB storage
pvesm add smb backup-smb \
  --server 192.168.1.200 \
  --share backup \
  --username backupuser \
  --password 'YourPassword'
```

### Pros:
- Windows compatible
- Easy to access

---

## Option 4: Proxmox Backup Server (PBS) - Best Option

### How to Install PBS

```bash
# Install PBS
apt update && apt install proxmox-backup-server -y

# Access PBS web UI
# https://your-pbs:8007
```

### Configure PBS in Proxmox

```bash
# Add PBS storage
pvesm add proxmox-backup-server pbs01 \
  --server 192.168.1.250 \
  --datastore main \
  --fingerprint YOUR-FINGERPRINT \
  --backup-username admin@pam
```

### Pros:
- Deduplication (saves space)
- Encryption available
- Incremental backup
- Provenance integrity

---

## Option 5: USB External Drive

### How to Configure

```bash
# List disks
lsblk

# Create partition
fdisk /dev/sdb

# Format
mkfs.ext4 /dev/sdb1

# Mount
mount /dev/sdb1 /mnt/usb

# Add as storage
pvesm add dir usb-backup \
  --path /mnt/usb \
  --content backup
```

### Pros:
- Offline backup possible
- Portable
- Complete isolation

---

## How to Choose Storage Type

| Storage | Best For | Speed | Safety |
|---------|---------|-------|--------|
| Local | Testing | Fast | Low |
| NFS | Small setups | Fast | Medium |
| SMB | Windows | Fast | Medium |
| PBS | Production | Fast | High |
| USB | Offline | Fast | High |
| Cloud | Offsite | Slow | High |

---

## Keywords

#storage #nfs #smb #pbs #how-to #beginner

## Related Articles

- [[Backup]]
- [[PBS]]
- [[Storage]]
---

[[index|Back to Proxmox VE]]
