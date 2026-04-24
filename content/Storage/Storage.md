# Storage Complete Guide

## Table of Contents

1. [Storage Types](#storage-types)
2. [Directory Storage](#directory-storage)
3. [LVM](#lvm)
4. [LVM-Thin](#lvm-thin)
5. [ZFS](#zfs)
6. [NFS](#nfs)
7. [iSCSI](#iscsi)
8. [Ceph/RADOS](#cephrados)
9. [Storage Configuration](#storage-configuration)
10. [Disk Performance](#disk-performance)
11. [Advanced Features](#advanced-features)
12. [Troubleshooting](#troubleshooting)

---

## Storage Types

### Overview

| Type | Snapshots | Clones | Compression | Deduplication | Performance |
|------|----------|--------|------------|--------------|------------|
| ZFS | Yes | Yes | Yes | Yes | Good |
| LVM-Thin | Yes | Yes | No | No | Good |
| Ceph | Yes | Yes | No | Yes | Medium |
| Directory | rsync | Copy | No | No | Medium |
| NFS | External | External | No | No | Varies |
| iSCSI | External | External | No | No | Varies |

---

## Directory Storage

### Overview

Directory storage uses the filesystem to store VM images and containers in a simple directory structure.

### Create via Web UI

1. **Datacenter** → **Storage** → **Add** → **Directory**
2. **ID**: `local-backup`
3. **Directory**: `/mnt/backup`
4. **Content**: ISO, Backup
5. **Enable**: Yes

### Create via CLI

```bash
# Create mount point
mkdir -p /mnt/backup

# Add storage
pvesm add dir local-backup \
  --path /mnt/backup \
  --content backup,iso,vztmpl
```

### Configuration

```bash
# Edit storage
pvesm set local-backup --path /mnt/backup

# Enable/disable
pvesm set local-backup --disable 0

# Set content types
pvesm set local-backup --content backup,iso,vztmpl
```

### Directory Structure

```
/mnt/backup/
├── template/
│   └── cache/
├── backup/
│   └── dump/
└── iso/
```

---

## LVM (Logical Volume Manager)

### Overview

LVM provides traditional block-level storage with snapshots and thin provisioning support.

### Create LVM

```bash
# Create PV
pvcreate /dev/sda

# Create VG
vgcreate pve /dev/sda

# Create LV
lvcreate -L 100G -n vmdata pve

# Add to Proxmox
pvesm add lvm pve-lvm \
  --vgname pve \
  --content rootdir,images
```

### LVM Commands

```bash
# List volumes
lvdisplay

# Extend LV
lvextend -L +50G /dev/pve/vmdata

# Reduce LV
lvreduce -L -20G /dev/pve/vmdata
```

---

## LVM-Thin

### Overview

LVM-thin provides thin provisioning, allowing overcommit and efficient snapshotting.

### Create LVM-Thin Pool

```bash
# Create thin pool
lvcreate -T -L 500G -n vmdata pve/vg00

# Verify
lvs -a
```

### Add to Proxmox

```bash
# Add LVM-thin storage
pvesm add lvmthin local-lvm \
  --vgname pve \
  --pool vmdata \
  --content rootdir,images
```

### Thin Pool Options

```bash
# Set pool options
pvesm set local-lvm --pool_size 500G

# Enable discard
pvesm set local-lvm --content rootdir,images
```

---

## ZFS

### Overview

ZFS provides advanced features including compression, deduplication, snapshots, and data integrity.

### Create ZFS Pool

```bash
# Single disk
zpool create -f local-zfs /dev/sda

# Mirror
zpool create -f mirror local-zfs /dev/sda /dev/sdb

# RaidZ
zpool create -f raidz1 local-zfs /dev/sda /dev/sdb /dev/sdc

# RaidZ2
zpool create -f raidz2 local-zfs /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

### ZFS Options

```bash
# Compression
zfs set compression=lz4 local-zfs

# Deduplication
zfs set dedup=compression local-zfs

# Checksum
zfs set checksum=on local-zfs

# Ashift (for 4K disks)
zpool create -o ashift=12 ...
```

### Add to Proxmox

```bash
# Add ZFS storage
pvesm add zfs local-zfs \
  --pool local-zfs \
  --content rootdir,images,snapshots
```

### ZFS Commands

```bash
# Status
zpool status
zpool list

# Datasets
zfs list
zfs list -t all

# Create dataset
zfs create local-zfs/vmdata

# Snapshot
zfs snapshot local-zfs/vmdata@backup-2024-01-01

# Clone
zfs clone local-zfs/vmdata@backup-2024-01-01 local-zfs/vmdata-clone
```

### ZFS Properties

```bash
# List properties
zfs get all local-zfs

# Set compression
zfs set compression=lz4 local-zfs
zfs set compression=zstd local-zfs

# Set dedup
zfs set dedup=off local-zfs
zfs set dedup=sha256 local-zfs
zfs set dedup=compression local-zfs

# Set atime
zfs set atime=off local-zfs

# Record size
zfs set recordsize=1M local-zfs
```

### ZFS Send/Receive

```bash
# Full send
zfs send local-zfs/vmdata@backup-2024-01-01 | ssh user@backup zfs receive -F backup/vmdata

# Incremental
zfs send -i local-zfs/vmdata@backup-2024-01-01 local-zfs/vmdata@backup-2024-01-02 | ssh user@backup zfs receive backup/vmdata
```

---

## NFS

### Overview

NFS (Network File System) provides network-accessible storage for backups, ISOs, and templates.

### Server Setup

```bash
# Install NFS server
apt install nfs-kernel-server -y

# Create export
mkdir -p /mnt/nfs
echo "/mnt/nfs 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)" >> /etc/exports

# Apply
exportfs -ra

# Start service
systemctl enable nfs-server
systemctl start nfs-server
```

### Add NFS to Proxmox

```bash
# Add NFS storage
pvesm add nfs nas01 \
  --server 192.168.1.50 \
  --export /mnt/nfs \
  --content iso,vztmpl,backup,images
```

### NFS Options

```bash
# Set options
pvesm set nas01 --options vers=4

# Mount options
pvesm set nas01 --mount-options "rsize=8192,wsize=8192"
```

### Mount NFS Manually

```bash
# Mount
mount -t nfs 192.168.1.50:/mnt/nfs /mnt/nfs

# With options
mount -t nfs -o rsize=8192,wsize=8192,vers=4 192.168.1.50:/mnt/nfs /mnt/nfs

# Persistent
# /etc/fstab
192.168.1.50:/mnt/nfs /mnt/nfs nfs defaults 0 0
```

---

## iSCSI

### Overview

iSCSI provides block-level access over network for high-performance storage.

### iSCSI Target

```bash
# Install target
apt install tgt -y

# Configure target
# /etc/tgt/targets.conf
target iqn.2024-01.local.nas:storage
    backing-store /dev/sda
    initiator-address 192.168.1.100
```

### Discover iSCSI

```bash
# Install initiator
apt install open-iscsi -y

# Discover targets
iscsiadm -m discovery -t sendtargets -p 192.168.1.50

# Login
iscsiadm -m node -T iqn.2024-01.local.nas:storage -p 192.168.1.50 --login

# Verify
iscsiadm -m session
```

### Add iSCSI to Proxmox

```bash
# Add iSCSI
pvesm add iscsi iscsi01 \
  --portal 192.168.1.50:3260 \
  --target iqn.2024-01.local.nas:storage \
  --content images
```

### LVM on iSCSI

```bash
# Create PV
pvcreate /dev/sdb

# Create VG
vgcreate iscsi-vg /dev/sdb

# Add LVM-thin
pvesm add lvmthin iscsi-lvm \
  --vgname iscsi-vg \
  --content images
```

---

## Ceph/RADOS

### Overview

Ceph provides scalable, distributed storage with built-in replication and erasure coding.

### Install Ceph

```bash
# Install
pveceph install

# Create mon
pveceph mon create

# Create OSD
pveceph osd create /dev/sda

# Create pool
pveceph pool create ceph-pool
```

### Ceph Architecture

```
┌─────────────────────────────────────┐
│           Ceph Clients              │
├─────────────────────────────────────┤
│            RADOS                    │
├─────���─��──────┬──────────────────────┤
│     OSDs    │     MONs           │
│   (storage)  │   (monitoring)     │
│    /dev/sdX  │   quorum         │
├──────────────┴──────────────────────┤
│         CRUSH Algorithm              │
└─────────────────────────────────────┘
```

### Add Ceph Storage

```bash
# Add CephFS
pvesm add cephfs cephfs \
  --mon 192.168.1.101:6789 \
  --user admin \
  --keyring /etc/pve/ceph.client.admin.keyring \
  --content rootdir,images
```

### Ceph Commands

```bash
# Status
ceph -s
ceph status

# OSDs
ceph osd status
ceph osd tree

# Pools
ceph pool ls
ceph pool ls detail

# PG
ceph pg ls
```

---

## Storage Configuration

### Add Storage (CLI)

```bash
# Directory
pvesm add dir local \
  --path /var/lib/vz \
  --content backup,iso,vztmpl

# LVM-thin
pvesm add lvmthin local-lvm \
  --vgname pve \
  --pool vmdata

# ZFS
pvesm add zfs local-zfs \
  --pool local-zfs

# NFS
pvesm add nfs nas01 \
  --server 192.168.1.50 \
  --export /mnt/nfs

# iSCSI
pvesm add iscsi iscsi01 \
  --portal 192.168.1.50 \
  --target iqn.target
```

### Storage Migration

```bash
# Migrate VM storage
qm migrate 100 pve2 --storage local2

# Move storage
pvesm move local:vm-100-disk-0 local2
```

### Storage Status

```bash
# List storage
pvesm status

# List storage with content
pvesm list

# Storage usage
df -h
```

---

## Disk Performance

### SSD vs HDD

```bash
# Check disk type
lsblk -o NAME,ROTA

# ROTA=0: SSD
# ROTA=1: HDD
```

### Benchmark

```bash
# Read speed
hdparm -t /dev/sda

# Write speed
dd if=/dev/zero of=/tmp/test bs=1M count=1000 oflag=direct

# Random I/O
fio --name=randwrite --ioengine=libaio --bs=4k --direct=1 --size=1G --numjobs=4 --runtime=30
```

### Performance Tips

- Use SSD for VM storage
- Enable discard for thin provisioning
- Use VirtIO drivers
- Enable IO thread
- Use proper cache mode

### NVMe Optimization

```bash
# Enable NVMe
# Use AHCI or VirtIO
# Set queue depth
echo 2048 > /sys/block/nvme0n1/queue/nr_requests
echo 0 > /sys/block/nvme0n1/queue/rotational
```

---

## Advanced Features

### Storage Clusters

```bash
# Add shared storage
pvesm add nfs shared-nfs \
  --server 192.168.1.50 \
  --export /mnt/shared \
  --content rootdir,images \
  --nodes pve1,pve2,pve3
```

### Backup Storage

```bash
# Dedicated backup storage
pvesm add dir backup \
  --path /mnt/backup \
  --content backup
  
# NFS backup
pvesm add nfs backup-nfs \
  --server 192.168.1.51 \
  --export /backups \
  --content backup
```

### Storage Replication

```bash
# Enable replication
pvesm start <storage-id>

# Or use PBS (Proxmox Backup Server)
```

---

## Troubleshooting

### No Space

```bash
# Check
pvesm status

# Clean backups
rm -rf /mnt/backup/dump/*

# Prune snapshots
zfs list -t snapshot
zfs destroy <snapshot>
```

### Slow Performance

```bash
# Check I/O
iostat -x 1

# Check for fragmentation
# ZFS
zpool list -v

# LVM
lvdisplay --maps
```

### Mount Issues

```bash
# Check mounts
mount | grep

# NFS
showmount -e 192.168.1.50

# iSCSI
iscsiadm -m session
```

---

## Keywords

#storage #zfs #lvm #nfs #iscsi #ceph #disk #ssd #nvme #thin-provisioning #compression #deduplication #snapshot #clone

## Links

- [[Proxmox-VE]] - Main documentation
- [[Installation]] - Initial storage setup
- [[Backup]] - Backup configuration
- [[VM]] - VM disk types
- [[Containers]] - Container storage
- [[Cluster]] - Shared storage