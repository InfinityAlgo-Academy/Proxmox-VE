---
title: Installation Guide
tags:
  - installation
  - setup
  - getting-started
  - guide
---

# Installation Guide

## Overview

This guide covers installing Proxmox VE from ISO on bare metal or as a virtual machine.

## System Requirements

### Minimum

| Component | Minimum |
|-----------|---------|
| CPU | 64-bit x86 |
| RAM | 4 GB |
| Storage | 32 GB |
| Network | 1 Gbps |

### Recommended

| Component | Recommended |
|-----------|------------|
| CPU | Multi-core (Intel VT-x/AMD-V) |
| RAM | 8 GB+ |
| Storage | 128 GB SSD |
| Network | 10 Gbps |

## Installation Methods

### Method 1: USB Media

1. **Download ISO**
   ```
   https://www.proxmox.com/en/downloads
   ```

2. **Create Boot USB**
   ```bash
   # Linux
   dd if=proxmox-ve_8.0.iso of=/dev/sdX status=progress
   
   # Rufus (Windows)
   # Select ISO, partition: GPT, filesystem: FAT32
   ```

3. **Boot & Install**
   - Boot from USB
   - Select "Install Proxmox VE"
   - Follow wizard

### Method 2: Network (PXE)

1. **Setup PXE Server**
   ```bash
   apt install isc-dhcp-server tftp-hpa pxelinux syslinux
   ```

2. **Configure PXE**
   ```
   /tftpboot/pxelinux.cfg/default:
   LABEL proxmox
       MENU LABEL Proxmox VE 8.0
       KERNEL proxmox/vmlinuz
       APPEND initrd=proxmox/initrd.img
   ```

### Method 3: Virtual Environment

#### Proxmox in VMware
```bash
# Create VM with:
- CPU: 2+ cores (VT-x)
- RAM: 4GB+
- Disk: 32GB+
- Network: Bridged
```

#### Proxmox in Nested ESXi
```bash
# Enable nested virtualization
vmware -v
# Or in VM settings:
# CPU > Hardware virtualization: Yes
```

## Post-Installation

### 1. Update System

```bash
# Update
apt update && apt full-upgrade -y

# Remove enterprise repo
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Add community repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > \
  /etc/apt/sources.list.d/pve-community.list
apt update
```

### 2. Set Timezone

```bash
timedatectl set-timezone Region/City
```

### 3. Configure Network

```bash
# Edit /etc/network/interfaces
nano /etc/network/interfaces
```

### 4. Create Storage

```bash
# Add local storage
pvesm add dir local --path /var/lib/vz --content vztmpl,iso,rootdir,backup
```

## First VM

```bash
# Create VM
qm create 100 \
  --name "first-vm" \
  --memory 2048 \
  --cores 2 \
  --net0 virtio,bridge=vmbr0 \
  --scsi0 local-lvm:32

# Start
qm start 100
```

## First Container

```bash
# Download template
pveam update
pveam available
pveam download debian-12-standard debian-12-standard_12.0.0-amd64.tar.zst

# Create
pct create 200 local:vztmpl/debian-12-standard_12.0.0-amd64.tar.zst \
  --rootfs local:8 \
  --memory 1024 \
  --cores 1

# Start
pct start 200
```

## Troubleshooting

- **Boot issues**: Check BIOS boot order
- **No network**: Check bridge configuration
- **Storage missing**: Add storage pool

## See Also

- [[index|Back to Proxmox VE]]
- [[Quick-Commands]]
- [[Host-Configuration]]
- [[Storage]]
- [[Networking]]