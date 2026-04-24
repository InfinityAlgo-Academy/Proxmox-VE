---
title: How to Restore from Backup - Complete Beginner's Guide
---

# How to Restore from Backup - Complete Beginner's Guide

## Question: How do I restore a VM or container from backup?

### Answer: Follow these simple steps

---

## How to Restore Using Web UI

### Step-by-Step:

**1. Navigate to Backup Storage**
- Go to **Datacenter** → **Storage** → Select your backup storage (e.g., local)
- Click **Content**

**2. Find Your Backup**
- Look for your backup file (format: `vzdump-qemu-100-DATE.tar.gz`)
- Click on the file

**3. Click Restore**
- Click **Restore** button
- Enter new VM ID or use original
- Click **Restore**

**4. Wait for Restore**
- Progress shows in task log
- VM appears in list when complete

---

## How to Restore Using CLI

### Restore a VM (QEMU)

```bash
# First, list available backups
vzdump --list

# Restore VM to new ID
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2024_01_01.tar.gz 200

# Or restore to original VM ID
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2024_01_01.tar.gz 100
```

### Restore a Container (LXC)

```bash
# List container backups
vzdump --list --vmid lxc

# Restore container
pct restore 200 /var/lib/vz/dump/vzdump-lxc-100-2024.tar.gz
```

---

## How to Restore to Different Storage

```bash
# Backup stored on one storage, restore to another
qmrestore /mnt/backup/vzdump-qemu-100-2024.tar.gz 100 \
  --storage local-lvm
```

---

## How to Restore Using PBS

```bash
# List PBS backups
proxmox-backup-client list snapshots pbs@pam:

# Restore from PBS
proxmox-backup-client restore pbs@pam:datastore/vzdump-qemu-100-2024 vm-100
```

---

## How to Verify Backup is Valid

```bash
# Check backup integrity
vzdump --vmid 100 --verify

# Or check the file
tar -tzf /var/lib/vz/dump/vzdump-qemu-100-2024.tar.gz | head
```

---

## Troubleshooting Restore Issues

### Error: "VM already exists"
```bash
# Use different VM ID
qmrestore backup.tar.gz 201
```

### Error: "Storage not available"
```bash
# Check storage is available
pvesm status
```

### Error: "Insufficient permissions"
```bash
# Check user permissions
pveum acl list /vms/100
```

---

## How to Restore Entire Cluster

### Step 1: Reinstall Proxmox
Follow standard install procedure

### Step 2: Restore Configuration
```bash
# Restore /etc/pve/
tar -xzf /backup/pve-config.tar.gz -C /
```

### Step 3: Restore VMs
```bash
# Restore each VM
qmrestore /backup/vm-100.tar.gz 100
qmrestore /backup/vm-101.tar.gz 101
```

---

## Pro Tips

1. **Always test restore** on a test VM first
2. **Keep backup log** for reference
3. **Document** your backup IDs

---

## Keywords

#restore #how-to #beginner #vzdump #recovery

## Related Articles

- [[Backup]]
- [[How-To-Configure-Backup]]
- [[Disaster-Recovery]]
---

[[index|Back to Proxmox VE]]
