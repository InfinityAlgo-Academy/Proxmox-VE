---
title: Complete VM Management
tags:
  - vm
  - virtual-machine
  - kvm
  - management
  - guide
---

# Complete VM Management

## Overview

Everything you need to know about managing KVM virtual machines in Proxmox VE.

## Create VM

### Basic Creation

```bash
# Create VM
qm create 100 \
  --name "web-server" \
  --memory 4096 \
  --cores 2 \
  --cpu host \
  --net0 virtio,bridge=vmbr0 \
  --scsi0 local-lvm:32
```

### With Full Options

```bash
qm create 100 \
  --name "production-vm" \
  --memory 8192 \
  --cores 4 \
  --cpu host \
  --numa 1 \
  --ostype l26 \
  --os-variant auto \
  --scsi0 local-lvm:64,discard=1,ssd=1,iothread=1 \
  --net0 virtio,bridge=vmbr0,queues=4,firewall=1 \
  --boot order=scsi0 \
  --bios ovmf \
  --efidisk0 local-lvm:1M,efitype=4g \
  --onboot 1 \
  --startup 1 \
  --description "Production web server"
```

## VM Configuration

### Hardware Options

| Option | Description | Example |
|--------|------------|---------|
| `--memory` | RAM in MB | `--memory 8192` |
| `--cores` | CPU cores | `--cores 4` |
| `--cpu` | CPU type | `--cpu host` |
| `--numa` | NUMA | `--numa 1` |
| `--scsi0` | Disk | `--scsi0 local:32G` |
| `--net0` | Network | `--net0 virtio,bridge=vmbr0` |

### CPU Configuration

```bash
# Set CPU type
qm set 100 --cpu host

# Set cores
qm set 100 --cores 4

# Enable NUMA
qm set 100 --numa 1

# CPU pinning
qm set 100 --cpuset 0-3
```

### Memory Configuration

```bash
# Set memory
qm set 100 --memory 8192

# Balloon (dynamic)
qm set 100 --balloon 2048

# Disable balloon
qm set 100 --balloon 0

# Huge pages
qm set 100 --hugepages 1024
```

### Disk Configuration

```bash
# Add disk
qm set 100 --scsi1 local-lvm:32G

# Resize disk
qm resize 100 scsi0 +10G

# Enable discard
qm set 100 --scsi0 local:,discard=1,ssd=1
```

### Network Configuration

```bash
# Set network
qm set 100 --net0 virtio,bridge=vmbr0

# Multi-queue
qm set 100 --net0 virtio,bridge=vmbr0,queues=4

# VLAN
qm set 100 --net0 virtio,bridge=vmbr0,tag=100

# Firewall
qm set 100 --net0 virtio,bridge=vmbr0,firewall=1
```

## VM Lifecycle

### Start/Stop

```bash
# Start
qm start 100

# Stop
qm stop 100

# Shutdown (graceful)
qm shutdown 100

# Reset
qm reset 100

# Reboot
qm reboot 100
```

### Console Access

```bash
# Text console
qm console 100

# SPICE
qm start 100 --spice

# Monitor
qm monitor 100
```

## Cloning & Templates

### Clone VM

```bash
# Full clone
qm clone 100 101 --full --name "clone-vm"

# Linked clone
qm clone 100 102 --name "linked-clone"

# Clone to template
qm template 100
```

### Convert to Template

```bash
qm template 100
```

## Snapshots

### Create Snapshot

```bash
# Create snapshot
qm snapshot 100 pre-update --description "Before update"

# With memory
qm snapshot 100 pre-update --description "With memory" --vmstate 1
```

### List Snapshots

```bash
qm snapshot list 100
```

### Restore Snapshot

```qm snapshot rollback 100 pre-update
```

### Delete Snapshot

```bash
qm snapshot delete 100 pre-update
```

## Migration

### Live Migration

```bash
# Live migrate
qm migrate 100 node2 --online
```

### Offline Migration

```bash
# Offline migrate
qm migrate 100 node2 --offline
```

## Backup

### Manual Backup

```bash
# Snapshot mode
vzdump --mode snapshot --vmid 100 --storage local

# Suspend mode
vzdump --mode suspend --vmid 100 --storage local

# Stop mode
vzdump --mode stop --vmid 100 --storage local
```

### Restore

```bash
qmrestore /var/lib/vz/dump/vzdump-qemu-100.tar.gz 100
```

## Troubleshooting

### VM Won't Start

```bash
# Check status
qm status 100

# Show boot info
qm show_boot_info 100

# Monitor
qm monitor 100
info block
```

### Performance Issues

- Increase CPU cores
- Increase memory
- Enable NUMA
- Use VirtIO drivers
- Enable iothread

## See Also

- [[index|Back to Proxmox VE]]
- [[VM-Templates]]
- [[Cloning]]
- [[VM-Migration-Live]]
- [[Snapshot-Management]]
- [[Backup]]