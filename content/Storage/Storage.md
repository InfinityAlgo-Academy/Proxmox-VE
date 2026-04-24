---
title: Complete Storage Guide
tags:
  - storage
  - lvm
  - zfs
  - nfs
  - iscsi
  - guide
---

# Complete Storage Guide

## Overview

Proxmox VE supports multiple storage types: local (LVM, ZFS, directory), network (NFS, iSCSI, Ceph), and cloud storage.

## Storage Types

| Type | Protocol | Features | Best For |
|------|----------|----------|----------|
| LVM-Thin | Block | Thin prov, snapshots | VM disks |
| ZFS | Block/Filesystem | Compression, dedup | All storage |
| Directory | Filesystem | Simple | ISO, backups |
| NFS | Network | Shared | Backups, templates |
| iSCSI | Block | Enterprise | Production |
| Ceph | Object/Block | Distributed | Enterprise HA |

## Local Storage

### Directory Storage

```bash
# Add directory
pvesm add dir local \
  --path /var/lib/vz \
  --content vztmpl,iso,rootdir,backup

# Custom path
pvesm add dir storage2 \
  --path /mnt/data \
  --content rootdir,backup
```

### LVM Thin

```bash
# Create LVM thin pool
lvcreate -L 500G --thinpool pool1 vg_data

# Add to Proxmox
pvesm add lvmthin pool1 \
  --vgname vg_data \
  --content rootdir,images

# Resize
lvresize -L +100G vg_data/pool1
```

### ZFS

```bash
# Create ZFS pool
zpool create -f mirror pool1 /dev/sda /dev/sdb

# ZFS options
zfs set compression=lz4 pool1
zfs set atime=off pool1
zfs set checksum=sha256 pool1

# Add ZFS
pvesm add zfspool pool1 \
  --pool pool1 \
  --content rootdir,images
```

## Network Storage

### NFS

```bash
# Install NFS server (on remote)
apt install nfs-kernel-server -y

# Export
echo "/mnt/share 192.168.1.0/24(rw,sync,no_subtree_check)" >> /etc/exports
exportfs -a

# Add NFS on Proxmox
pvesm add nfs nfs-backup \
  --server 192.168.1.200 \
  --export /mnt/share \
  --content backup,vztmpl,iso
```

### iSCSI

```bash
# Add iSCSI
pvesm add iscsi iscsi-store \
  --portal 192.168.1.200:3260 \
  --target iqn.2024-01.com.example:store \
  --content images,rootdir

# Use LUN
pvesm add block iscsi-block \
  --target /dev/sdc \
  --content images
```

### CIFS/SMB

```bash
# Add SMB/CIFS
pvesm add cifs smb-share \
  --server 192.168.1.200 \
  --share backup \
  --username backup \
  --password secret \
  --content backup,iso,vztmpl
```

## Storage Commands

| Command | Description |
|---------|-------------|
| `pvesm status` | Show status |
| `pvesm list` | List storage |
| `pvesm add` | Add storage |
| `pvesm remove` | Remove |
| `pvesmscan` | Scan |

## Disk Management

### Add Physical Disk

```bash
# Create partition
fdisk /dev/sdb
# n, p, w

# Create PV
pvcreate /dev/sdb1

# Create VG
vgcreate vg_data /dev/sdb1
```

### LVM Commands

```bash
# Create LV
lvcreate -L 100G -n data vg_data

# Resize
lvresize -L +50G /dev/vg_data/data

# Remove
lvremove vg_data/data
```

### ZFS Commands

```bash
# Create snapshot
zfs snapshot pool1/vm-100@pre-update

# Rollback
zfs rollback pool1/vm-100@pre-update

# Clone
zfs clone pool1/vm-100@pre-update pool1/vm-100-clone

# Send/Receive
zfs send pool1/vm-100@backup | zfs receive pool2/vm-100
```

## Storage for VMs

### VM Disk Options

```bash
# VirtIO SCSI (recommended)
qm set 100 --scsi0 local-lvm:32,discard=1,ssd=1

# SATA
qm set 100 --sata0 local-lvm:32

# IDE (legacy)
qm set 100 --ide0 local-lvm:32
```

### Disk Performance

| Type | Performance | Use |
|------|------------|-----|
| VirtIO SCSi | Best | Production |
| VirtIO Block | Good | General |
| SATA | Good | Legacy OS |
| IDE | Poor | Legacy |

## Backup Storage

```bash
# Dedicated backup storage
pvesm add dir backup \
  --path /mnt/backup \
  --content backup \
  --maxfiles 7 \
  --prune-daily 1

# NFS backup
pvesm add nfs nfs-backup \
  --server 192.168.1.200 \
  --export /backup \
  --content backup
```

## Storage Optimization

### LVM Thin

```bash
# Enable thin provisioning
lvcreate -L 500G --thinpool thin_pool vg_data

# Set autoextend
lvm.conf {
  monitoring = 1
  autoextend = 1
  percent = 20
}
```

### ZFS Optimization

```bash
# ARC size
echo "options zfs arc_max=4294967296" >> /etc/modprobe.d/zfs.conf

# L2ARC (SSD cache)
zpool add pool1 cache /dev/sdc

# ZIL (SLOG)
zpool add pool1 log /dev/sdd
```

## Troubleshooting

### Storage Offline

```bash
# Check status
pvesm status

# Scan
pvesm scan-lvm /dev/sda

# Rescan
pvesm scan
```

### Performance

- Use SSD for ZFS pool
- Enable compression
- Use VirtIO drivers
- Enable discard

## See Also

- [[index|Back to Proxmox VE]]
- [[LVM-Management]]
- [[ZFS-Tuning]]
- [[NFS]]
- [[iSCSI]]
- [[Backup]]