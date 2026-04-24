---
title: Proxmox VE
---

# PROXMOX VE
# Complete Virtualization Platform

---

## Overview

Proxmox VE is an open-source virtualization platform that provides enterprise-class features including KVM virtual machines, LXC containers, software-defined storage, and high-availability clustering.

---

## System Requirements

### Hardware Requirements

| Component     | Minimum    | Recommended             | Production           |
| ------------- | ---------- | ----------------------- | -------------------- |
| CPU           | 64-bit x86 | 64-bit x86 (VT-x/AMD-V) | Multi-core Xeon/EPYC |
| RAM           | 4 GB       | 8 GB                    | 32 GB+               |
| System Disk   | 32 GB      | 64 GB SSD               | 128 GB SSD RAID1     |
| Network       | 1 Gbps     | 10 Gbps                 | 10 Gbps bonded       |
| Memory per VM | 512 MB     | 2 GB                    | Depends on workload  |

### Software Requirements

| Component | Version |
|-----------|---------|
| Proxmox VE | 7.0+ / 8.0+ |
| CPU Support | Intel VT-x / AMD-V |
| Boot | UEFI / Legacy BIOS |

---

## Installation

### Installation Methods

1. **ISO Installation** - Standard USB/CD installation
2. **Network PXE** - Network-based deployment
3. **Vagrant** - Development/testing environments

### Post-Installation Steps

```bash
# 1. Update system
apt update && apt full-upgrade -y

# 2. Configure repositories
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > \
  /etc/apt/sources.list.d/pve-community.list
apt update

# 3. Set timezone
timedatectl set-timezone Region/City

# 4. Install useful packages
apt install -y curl wget git htop iftop iotop net-tools tcpdump
```

---

## Architecture

### Virtualization Types

| Type | Technology | Use Case |
|------|-----------|----------|
| Full Virtualization | KVM/QEMU | Windows, BSD, complex Linux |
| Container Virtualization | LXC | Linux workloads, lightweight apps |
| Nested Virtualization | KVM in KVM | Testing, development |

### Cluster Architecture

```
+------------------+
|   Proxmox VE    |
+------------------+
       |
  +----+----+
  |         |
Node 1   Node 2   Node 3
  |    \   |
  |     +--+--+
  |     |      |
VM 100  VM 101  VM 102
```

---

## Networking

### Network Types

| Type | Description | Use Case |
|------|-------------|----------|
| Bridge | Linux bridge | Standard VM networking |
| Bond | Link aggregation | Redundancy, performance |
| VLAN | 802.1q tagging | Network segmentation |
| OVS | Open vSwitch | Advanced networking |
| VXLAN | L2 over L3 | Multi-site |

### Network Configuration

```bash
# Create bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

### VLAN Configuration

```bash
# Create VLAN
ip link add link vmbr0 name vmbr0.100 type vlan id 100
ip addr add 192.168.100.1/24 dev vmbr0.100
ip link set vmbr0.100 up
```

### Network Bond Modes

| Mode | Description | Redundancy | Performance |
|------|-------------|------------|-------------|
| active-backup | Primary/backup | Yes | No |
| balance-rr | Round-robin | No | Yes |
| balance-xor | Hash-based | No | Yes |
| 802.3ad (LACP) | IEEE 802.3ad | Yes | Yes |
| balance-tlb | Load balancing | Yes | Yes |

---

## Storage

### Storage Types

| Type | Protocol | Features | Best For |
|------|----------|----------|----------|
| LVM-Thin | Block | Thin provisioning, snapshots | VM disks |
| ZFS | Block/Filesystem | Compression, deduplication, RAID | All storage |
| NFS | Network | Shared access | ISO, backups, templates |
| iSCSI | Block | Block-level | Enterprise storage |
| Ceph | Object/Block | Distributed, replicated | Enterprise HA |
| Directory | Filesystem | Simple | Local storage |

### Storage Commands

```bash
# Add directory storage
pvesm add dir local --path /var/lib/vz \
  --content vztmpl,iso,rootdir,backup

# Add NFS
pvesm add nfs nfs-backup \
  --server 192.168.1.200 \
  --export /mnt/backup \
  --content backup,vztmpl,iso

# Add ZFS pool
zpool create -f mirror pool1 /dev/sda /dev/sdb
zfs set compression=lz4 pool1
zfs set atime=off pool1
```

### ZFS Configuration

```bash
# Create mirror pool
zpool create -f mirror pool1 /dev/sda /dev/sdb

# Create RAIDZ
zpool create -f raidz2 pool2 /dev/sda /dev/sdb /dev/sdc /dev/sdd

# Set properties
zfs set compression=lz4 pool1
zfs set dedup=lz4 pool1
zfs set atime=off pool1
zfs set checksum=sha256 pool1
```

---

## Virtual Machines

### Create VM

```bash
# Full configuration
qm create 100 \
  --name "production-server" \
  --memory 8192 \
  --cores 4 \
  --cpu host \
  --numa 1 \
  --scsi0 local-lvm:64,discard=1,ssd=1,iothread=1 \
  --net0 virtio,bridge=vmbr0,queues=4,macaddr=XX:XX:XX:XX:XX:XX \
  --boot order=scsi0 \
  --bios ovmf \
  --efidisk0 local-lvm:1M,efitype=4g \
  --onboot 1 \
  --startup 1 \
  --description "Production Web Server"
```

### VM Hardware Options

| Hardware | Parameter | Example |
|----------|-----------|----------|
| CPU | --cores | 2-32 cores |
| Memory | --memory | 512-128GB |
| CPU Type | --cpu | host, Penryn, Haswell |
| Disk | --scsi0 | local:32G |
| Network | --net0 | virtio, e1000, rtl8139 |
| GPU | --hostpci0 | 01:00,pcie=1 |
| USB | --usb0 | host=046d:c52b |

### VM Lifecycle Management

```bash
# Start/Stop/Restart
qm start 100
qm stop 100
qm shutdown 100
qm reset 100
qm reboot 100

# Console access
qm console 100
qm monitor 100

# Clone and template
qm template 100
qm clone 100 101 --full --name "web-clone"

# Migrate
qm migrate 100 pve2 --online
qm migrate 100 pve2 --offline
```

### VM Backup

```bash
# Snapshot mode (live)
vzdump --mode snapshot --vmid 100 --storage local --compress zstd

# Suspend mode
vzdump --mode suspend --vmid 100 --storage local

# Stop mode
vzdump --mode stop --vmid 100 --storage local
```

---

## Containers (LXC)

### Create Container

```bash
pct create 200 local:vztmpl/debian-12-standard.tar.gz \
  --rootfs local:8 \
  --memory 2048 \
  --cores 2 \
  --hostname web-server \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --onboot 1 \
  --unprivileged 1 \
  --features keyctl=1,nesting=1
```

### Container Configuration

```bash
# Start/Stop
pct start 200
pct stop 200

# Console
pct console 200

# Execute commands
pct exec 200 -- apt update
pct exec 200 -- systemctl nginx start

# Resources
pct set 200 --memory 4096
pct set 200 --cores 4
```

### Container Features

| Feature | Parameter | Description |
|---------|-----------|-------------|
| Nesting | --features nesting=1 | Run containers in container |
| Keyctl | --features keyctl=1 | Keyring access |
| FUSE | --features fuse=1 | FUSE filesystem |
| Unprivileged | --unprivileged 1 | Non-root UID mapping |

---

## Security

### Firewall Configuration

```bash
# Install UFW
apt install ufw -y

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Allow Proxmox
ufw allow 8006/tcp comment 'Proxmox Web'
ufw allow 22/tcp comment 'SSH'

# Enable
ufw enable
```

### User Management

```bash
# Create user
pveum user add admin@example.com --password "SecurePassword"

# Create group
pveum group add administrators

# Add user to group
pveum group adduser administrators admin@example.com

# Grant permissions
pveum acl modify /vms/ --user admin@example.com --role PVEAdmin
pveum acl modify /storage/ --user admin@example.com --role PVEAudit

# API tokens
pveum user token add admin@example.com automation
```

### Roles and Permissions

| Role | Description |
|------|-------------|
| PVEAudit | Read-only access |
| PVEUser | Standard user |
| PVEPowerAdmin | Power user |
| PVEAdmin | Full administrator |

---

## Clustering

### Create Cluster

```bash
# First node
pvecm create production-cluster

# Add nodes
pvecm add 192.168.1.101
pvecm add 192.168.1.102

# Verify
pvecm status
```

### High Availability

```bash
# Create HA group
ha-manager group add production pve1 pve2 pve3

# Enable HA for VM
ha-manager enable vm:100 --group production

# Set priority
ha-manager crm add production --vm 100 --priority 100
```

### Quorum

```
Quorum requires: (N/2)+1 nodes

2 nodes = No quorum (need 3)
3 nodes = Quorum (2 needed)
4 nodes = Quorum (3 needed)
5 nodes = Quorum (3 needed)
```

---

## Backup and Recovery

### Backup Strategy (3-2-1 Rule)

- 3 copies of data
- 2 different storage types
- 1 offsite copy

### Backup Commands

```bash
# Manual backup
vzdump --mode snapshot --vmid 100 --storage local --compress zstd

# Schedule via Web UI
# Datacenter -> Backup -> Add

# Restore
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2024.tar.gz 100
pct restore 200 /var/lib/vz/dump/vzdump-lxc-200-2024.tar.gz
```

---

## Performance Optimization

### CPU Optimization

```bash
# Host CPU mode
qm set 100 --cpu host

# CPU pinning
qm set 100 --cpuset 0-3

# NUMA
qm set 100 --numa 1 --numa0 memory=4096,cpus=0-3
```

### Memory Optimization

```bash
# Disable balloon
qm set 100 --balloon 0

# Lock memory
qm set 100 --lockmemory 1

# Huge pages
qm set 100 --hugepages 1024
```

### Network Optimization

```bash
# VirtIO multi-queue
qm set 100 --net0 virtio,bridge=vmbr0,queues=4

# I/O thread
qm set 100 --iothread 1
```

### Disk Optimization

```bash
# VirtIO SCSI with discard
qm set 100 --scsi0 local:,discard=1,ssd=1,iothread=1
```

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| VM won't start | Check CPU/memory/storage availability |
| No network | Restart networking, check bridge |
| Storage offline | Verify mount, check connection |
| Web UI not loading | Check pveproxy service |
| High CPU | Reduce VM CPU allocation |
| Cannot access | Check firewall rules |

### Diagnostic Commands

```bash
# Node status
pvesh get /nodes/localhost/status

# Cluster status
pvecm status

# Storage
pvesm status

# VMs and containers
qm list
pct list

# Logs
journalctl -u pveproxy -n 100
journalctl -u pvedaemon -n 100

# Services
systemctl status pveproxy pvedaemon corosync
```

---

## Quick Reference

### Essential Commands

| Category | Command |
|----------|---------|
| VM List | qm list |
| VM Start | qm start 100 |
| VM Stop | qm stop 100 |
| VM Console | qm console 100 |
| CT List | pct list |
| CT Start | pct start 200 |
| CT Stop | pct stop 200 |
| Storage | pvesm status |
| Cluster | pvecm status |
| Node | pvesh get /nodes/localhost/status |

### Port Reference

| Port | Service |
|------|----------|
| 8006 | Proxmox Web UI (HTTPS) |
| 22 | SSH |
| 3128 | SPICE Proxy |
| 6000-6005 | SPICE Display |

---

## Resources

| Resource | URL |
|----------|-----|
| Documentation | https://pve.proxmox.com/wiki/ |
| Forum | https://forum.proxmox.com/ |
| Bug Tracker | https://bugzilla.proxmox.com/ |
| Downloads | https://www.proxmox.com/downloads/ |
| Training | https://www.proxmox.com/training/ |

---

**Version**: Proxmox VE 8.0+  
**Last Updated**: April 2025  
**License**: AGPL v3