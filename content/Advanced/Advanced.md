# Proxmox VE Advanced Guide

## Table of Contents

1. [Nested Virtualization](#nested-virtualization)
2. [SR-IOV](#sr-iov)
3. [vGPU Configuration](#vgpu-configuration)
4. [PCI Passthrough](#pci-passthrough)
5. [High Performance Tuning](#high-performance-tuning)
6. [Custom Networks](#custom-networks)
7. [Storage Advanced](#storage-advanced)
8. [Cluster Advanced](#cluster-advanced)
9. [Custom Installation](#custom-installation)
10. [API Advanced](#api-advanced)

---

## Nested Virtualization

### What is Nested Virtualization?

Running a hypervisor inside a virtual machine - useful for testing and learning.

### Enable Nested Virtualization

```bash
# Check if nested is enabled
# Intel
cat /sys/module/kvm_intel/parameters/nested
# AMD
cat /sys/module/kvm_amd/parameters/nested

# Enable nested (Intel)
echo "options kvm_intel nested=1" >> /etc/modprobe.d/kvm.conf

# Enable nested (AMD)
echo "options kvm_amd nested=1" >> /etc/modprobe.d/kvm.conf

# Reload module
modprobe -r kvm_intel
modprobe kvm_intel
```

### Configure LXC for Nested

```bash
# Enable nesting in container
pct set CTID --features nesting=1

# Full nested (Docker/Podman)
pct set CTID --features nesting=1,keyctl=1,fuse=1,shiftfs=1
```

### Proxmox in VM

```bash
# Create VM with nested support
qm create 9000 \
  --memory 8192 \
  --cores 4 \
  --cpu host \
  --numa 1 \
  --nested 1

# Install Proxmox in VM
# Follow standard installation
# Nested Proxmox can run:
# - Test environments
# - Learning labs
# - Development
```

### Nested Performance

```bash
# Optimize for nested
qm set 9000 --cpu host

# Enable hardware virtualization
qm set 9000 --arg2 +vmx
qm set 9000 --arg2 +svm
```

---

## SR-IOV

### What is SR-IOV?

Single Root I/O Virtualization - allows a PCIe device to appear as multiple separate PCIe devices.

### Requirements

- CPU with VT-d / AMD-Vi
- Motherboard + BIOS support
- Compatible PCIe device

### Enable SR-IOV

```bash
# Check GPU SR-IOV capability
lspci -vv | grep -i "SR-IOV"

# Enable in kernel
echo "options igbxf allow_unsupported=1" >> /etc/modprobe.d/igbxf.conf
```

### Configure Network SR-IOV

```bash
# Create virtual function
echo 2 > /sys/class/net/eth0/device/sriov_numvfs

# Verify
ip link show eth0

# Use in VM
qm set VMID --net0 virtio,bridge=vmbr0,macaddr=XX:XX:XX:XX:XX:XX
```

---

## vGPU Configuration

### NVIDIA vGPU

```bash
# Requirements:
# - NVIDIA GPU with vGPU support
# - NVIDIA license server
# - vGPU software

# Install vGPU Manager
wget <nvidia-driver>
./NVIDIA-vGPU_Manager.sh

# Configure vGPU
qm set VMID --hostpci0 01:00,pcie=1,vgpu=1
```

### Intel vGPU

```bash
# Enable Intel GPU passthrough
qm set VMID --hostpci0 00:02.0,pcie=1

# Allocate GPU memory
qm set VMID --hostpci0 00:02.0,pcie=1,mdev=512
```

### vGPU Profiles

| Profile | Memory | Use Case |
|---------|--------|----------|
| nvidia-grid-p40-4q | 4GB | Desktop |
| nvidia-grid-p40-8q | 8GB | Workstation |
| intel-gVTg-2.0 | 2GB | Basic |

---

## PCI Passthrough

### Full PCI Passthrough

```bash
# Enable IOMMU
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
update-grub
reboot

# Find device
lspci -nn
# 01:00.0 VGA compatible controller [0300]: NVIDIA Corporation [10de:1b80]

# Bind to VFIO
echo 0000:01:00.0 > /sys/bus/pci/drivers/vfio-pci/new_id
```

### Multiple GPU Passthrough

```bash
# Add GPU to VM
qm set VMID --hostpci0 01:00,pcie=1,x-vga=1

# Add second GPU
qm set VMID --hostpci1 02:00,pcie=1

# Add audio
qm set VMID --hostpci0 01:00.1,pcie=1
```

### USB Controller Passthrough

```bash
# List controllers
lspci | grep -i usb

# Add to VM
qm set VMID --usb0 xhci,pcie=1
```

### Network Card Passthrough

```bash
# Add network card
qm set VMID --net0 hostpci0=02:00,pcie=1
```

### Passthrough Configuration

```bash
# Configure
qm set VMID --hostpci0 01:00,pcie=1,x-vga=1,romfile=vgabios.bin

# Options:
# - pcie=1: PCIe
# - x-vga=1: Primary display
# - romfile: Custom VBIOS
```

---

## High Performance Tuning

### CPU Pinning

```bash
# Pin VM to specific cores
qm set VMID --cores 2 --cpuset 0-1

# Pin to NUMA node
qm set VMID --numa 1 --numa0 memory=4096,cpus=0-1
```

### Huge Pages

```bash
# Enable huge pages
qm set VMID --hugepages 1024

# Configure host
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

### I/O Thread

```bash
# Enable I/O thread
qm set VMID --scsi0 local:,iothread=1
```

### Network Multi-Queue

```bash
# Enable multi-queue
qm set VMID --net0 virtio,bridge=vmbr0,queues=4
```

### VirtIO Optimization

```bash
# Configure cache
qm set VMID --scsi0 local:,cache=none,iothread=1

# Enable discard
qm set VMID --scsi0 local:,discard=1,ssd=1
```

### Memory Tuning

```bash
# Lock memory
qm set VMID --lockmemory 1

# Disable balloon
qm set VMID --balloon 0
```

---

## Custom Networks

### OVS Switch

```bash
# Install Open vSwitch
apt install openvswitch-switch -y

# Create switch
ovs-vsctl add-br vmbr0

# Add ports
ovs-vsctl add-port vmbr0 eth0

# Configure VLAN
ovs-vsctl set port vmbr0 tag=100
```

### VXLAN

```bash
# Create VXLAN
ovs-vsctl add-br vmbr0
ovs-vsctl add-port vmbr0 vxlan0 -- set interface vxlan0 type=vxlan \
  option:remote_ip=192.168.1.101 \
  option:key=1000
```

### Geneve

```bash
# Create Geneve tunnel
ovs-vsctl add-port vmbr0 geneve0 -- set interface geneve0 type=geneve \
  option:key=flow \
  option:remote_ip=192.168.1.101
```

### Internal Network

```bash
# Isolated network
auto vmbr1
iface vmbr1 inet static
    address 10.0.0.1/24
    bridge-ports none
    bridge-stp off
```

### Hot-Plug Network

```bash
# Add network without reboot
qm set VMID --net1 virtio,bridge=vmbr1
```

---

## Storage Advanced

### Ceph Advanced

```bash
# Install Ceph
pveceph install

# Create monitor
pveceph mon create

# Create OSD
pveceph osd create /dev/sda

# Create pool
pveceph pool create ceph-pool

# Configure
ceph pool set ceph-pool size 3
ceph pool set ceph-pool min_size 2
```

### Multi-Path I/O

```bash
# Install multipath
apt install multipath-tools

# Configure
mpathconf --enable

# Check
multipath -ll
```

### LVM Advanced

```bash
# Create thin pool
lvcreate -T -L 500G -n vmdata pve/vg00

# Configure thin pool
lvchange --autactivate y pve/vg00

# Snapshot
lvcreate -s -n snap -L 10G pve/vg00/vmdata
```

### ZFS Advanced

```bash
# ZFS ARC tuning
echo 8589934592 > /sys/module/zfs/parameters/zfs_arc_max

# ZFS deduplication
zfs set dedup=compression pool/dataset

# ZFS special vdev
zpool add pool special mirror /dev/sdc /dev/sdd

# ZFS cache
zpool add pool cache /dev/sde
```

### NFS Advanced

```bash
# Mount with custom options
mount -t nfs -o hard,intr,rsize=524288,wsize=524288 192.168.1.50:/data /mnt/nfs

# Persistent mount
# /etc/fstab
192.168.1.50:/data /mnt/nfs nfs hard,intr,rsize=524288,wsize=524288 0 0
```

---

## Cluster Advanced

### Quorum Settings

```bash
# Check quorum
pvecm status

# Set expected votes
pvecm expected 3

# Configure
# /etc/pve/corosync.conf
totem {
    interface {
        bindnetaddr: 192.168.1.100
        mcastport: 5405
    }
}
```

### Fencing

```bash
# Install fence agents
apt install fence-agents-all

# Configure in Web UI
# Datacenter → HA → Fencing
```

### Migration Advanced

```bash
# Live migrate with bandwidth limit
qm migrate VMID target-node --online --bandwidth 100M

# Storage migration
qm migrate VMID target-node --storage target-storage
```

### HA Rules

```bash
# Create HA group
ha-manager group add production pve1 pve2 pve3

# Set VM priority
ha-manager crm add production --vm VMID --priority 100
```

---

## Custom Installation

### USB Installation

```bash
# Create bootable USB
dd if=proxmox-8.0.iso of=/dev/sdX bs=1M status=progress

# UEFI boot
# Format USB as FAT32
# Extract ISO contents
```

### Network Installation (PXE)

```bash
# Configure DHCP
# /etc/dhcp/dhcpd.conf
next-server 192.168.1.10;
filename "pxelinux.0";

# Install PXE server
apt install tftp-hpa

# Setup
mkdir /tftpboot
cp /usr/lib/syslinux/pxelinux.0 /tftpboot/
mkdir /tftpboot/pxelinux.cfg
```

### Automation Installation

```bash
# Use kickstart / preseed
# Download installation files
wget proxmox.preseed

# Configure
# Add to kernel parameters
preseed/url=http://server/preseed.cfg
```

### LVM on root

```bash
# During installation:
# Advanced → Configure LVM
# Create /boot ext4
# Create / as LVM thin pool
```

### ZFS on root

```bash
# Installation:
# Select "ZFS (Mirror)" or "ZFS"
# Choose disks
# Set configuration
# Boot
```

---

## API Advanced

### Batch Operations

```python
import requests

# Start multiple VMs
VM_IDS = [100, 101, 102]

for vmid in VM_IDS:
    url = f"https://pve:8006/api2/json/nodes/pve/qemu/{vmid}/status/start"
    requests.post(url, auth=('root@pam', 'password'))
```

### Webhook Integration

```bash
# Create webhook for VM events
# Use pveproxy events
tail -f /var/log/pveproxy/access.log
```

### Custom API Scripts

```bash
#!/bin/bash
# Custom backup script

VM_LIST="100 101 102"
DATE=$(date +%Y%m%d)

for VMID in $VM_LIST; do
  echo "Backing up VM $VMID..."
  vzdump $VMID --mode snapshot --storage local-backup
done
```

### Prometheus Exporter

```bash
# Export metrics
# Install node_exporter
# Configure to expose Proxmox metrics

# Use Prometheus federation
```

---

## Performance Comparison

### Network Performance

| Driver | Speed | CPU Usage |
|--------|-------|-----------|
| e1000 | 1 Gbps | High |
| RTL8139 | 100 Mbps | High |
| VirtIO | 10 Gbps | Low |

### Disk Performance

| Driver | Speed | Features |
|--------|-------|----------|
| IDE | 100 MB/s | Basic |
| SATA | 200 MB/s | Basic |
| SCSI | 300 MB/s | Good |
| VirtIO | 1 GB/s | Best |
| VirtIO + iothread | 1.5 GB/s | Best |

### CPU Performance

| Mode | Speed | Migration |
|------|-------|-----------|
| Default | 100% | Yes |
| Host | 100% | No |
| Passthrough | 100% | No |

---

## Quick Reference

### Advanced Commands

```bash
# CPU pinning
qm set VMID --cpuset 0-3

# Huge pages
qm set VMID --hugepages 1024

# I/O thread
qm set VMID --iothread 1

# VirtIO multi-queue
qm set VMID --queues 4

# PCI passthrough
qm set VMID --hostpci0 01:00,pcie=1
```

### Tuning Checklist

- [ ] Enable CPU host mode
- [ ] Use VirtIO drivers
- [ ] Enable I/O thread
- [ ] Enable network multi-queue
- [ ] Use huge pages for memory-intensive VMs
- [ ] Pin CPUs for real-time

## Keywords

#advanced #nested #sriov #vgpu #pci-passthrough #performance #tuning #api

## Links

- [[Proxmox-VE]] - Main documentation
- [[VM]] - VM management
- [[Cluster]] - Clustering
- [[Storage]] - Storage
- [[API]] - API documentation