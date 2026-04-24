# Virtual Machines (VM) Complete Guide

KVM (Kernel-Based Virtual Machine) in Proxmox VE provides full virtualization for running multiple operating systems with complete hardware isolation.

## Table of Contents

1. [What is KVM?](#what-is-kvm)
2. [KVM Architecture](#kvm-architecture)
3. [Creating VMs](#creating-vms)
4. [Disk Configuration](#disk-configuration)
5. [CPU Configuration](#cpu-configuration)
6. [Memory Configuration](#memory-configuration)
7. [Network Configuration](#network-configuration)
8. [GPU Passthrough](#gpu-passthrough)
9. [USB Passthrough](#usb-passthrough)
10. [VirtIO Drivers](#virtio-drivers)
11. [Cloud-Init](#cloud-init)
12. [Operations](#operations)
13. [Performance Tuning](#performance-tuning)
14. [Security](#security)
15. [Common Use Cases](#common-use-cases)

---

## What is KVM?

KVM (Kernel-Based Virtual Machine) is a full virtualization solution built into the Linux kernel, turning Linux into a Type-1 hypervisor.

### Key Characteristics

- **Type-1 Hypervisor**: Bare-metal virtualization
- **Hardware Virtualization**: Intel VT-x / AMD-V
- **Full Isolation**: Separate kernel for each VM
- **Any OS**: Windows, Linux, BSD, macOS
- **Near-Native Performance**: VirtIO drivers
- **No Licensing**: 100% open-source

### KVM vs LXC

| Feature | KVM VM | LXC Container |
|---------|-------|---------------|
| Kernel | Separate | Shared |
| OS Support | Any | Linux only |
| Isolation | Full | Namespace |
| Overhead | 2-5% | <1% |
| Boot Time | 10-30s | <1s |
| Resource Usage | Higher | Lower |

---

## KVM Architecture

### Virtualization Stack

```
┌─────────────────────────────────────────┐
│         Proxmox Web UI / CLI           │
├─────────────────────────────────────────┤
│              QEMU                      │
├─────────────────────────────────────────┤
│          KVM Hypervisor                 │
├──────────────┬────────────────────────┤
│   VirtIO     │    Emulated Devices     │
│  - Block     │    - e1000 NIC          │
│  - Network   │    - IDE/SCSI           │
│  - SCSI      │    - VGA                │
│  - Balloon   │    - USB               │
│  - 9PFS      │    - Serial           │
├──────────────┴────────────────────────┤
│         Linux Kernel                   │
├─────────────────────────────────────────┤
│      Hardware (CPU/RAM/Storage)       │
└─────────────────────────────────────────┘
```

### Hardware Support

```
┌─────────────────────────────────────────┐
│             VM Hardware                 │
├─────────────────────────────────────────┤
│  CPU: cores, sockets, flags, cputype   │
│  RAM: memory, balloon, huge pages     │
│  Disk: IDE, SCSI, SATA, VirtIO, NVMe  │
│  Network: e1000, VirtIO, vmxnet3     │
│  Video: Standard, VirtIO, GPU       │
│  USB: EHCI, XHCI, passthrough       │
│  Console: Serial, VNC, SPICE        │
│  TPM: 2.0                          │
│  EFI: Secure Boot                   │
└─────────────────────────────────────────┘
```

---

## Creating VMs

### Prerequisites

```bash
# Check virtualization support
egrep -c '(vmx|svm)' /proc/cpuinfo
# Should return > 0

# Load KVM modules
modprobe kvm_intel  # Intel
modprobe kvm_amd    # AMD

# Verify
lsmod | grep kvm
```

### Method 1: Web UI

1. **Navigate**: Create VM
2. **General**:
   - Name: `windows-server`
   - VM ID: `100`
3. **OS**:
   - Type: `Microsoft Windows`
   - Version: `11/2022/10/8/7`
4. **CD/DVD**:
   - ISO: `local:iso/windows-11.iso`
   - Or use: `CD/DVD Drive: ide`
5. **System**:
   - Machine: `Q35` (recommended)
   - BIOS: `OVMF` (UEFI) or `SeaBIOS`
   - Add EFI Disk: Checked
   - TPM: Choose if needed
6. **Hard Disk**:
   - Bus: `VirtIO Block` or `SCSI`
   - Storage: `local`
   - Size: `64GB`
   - Cache: `Write-through`
7. **CPU**:
   - Sockets: `1`
   - Cores: `4`
   - Type: `x86-64v2`
8. **Memory**:
   - Memory (MiB): `8192`
   - Balloon: Check (optional)
9. **Network**:
   - Bridge: `vmbr0`
   - Model: `VirtIO` (paravirtualized)
10. **Confirm**: Create

### Method 2: CLI

```bash
# Create VM
qm create 100 \
  --name "windows-server" \
  --memory 8192 \
  --cores 4 \
  --sockets 1 \
  --cpu host \
  --ostype win10 \
  --scsi0 local:vm-100-disk-0,size=64G,discard=1,ssd=1 \
  --net0 virtio,bridge=vmbr0,macaddr=BC:24:11:01:01:01 \
  --cdrom local:iso/windows-11.iso \
  --bios ovmf \
  --machine q35 \
  --onboot 1

# Start VM
qm start 100
```

### Method 3: Import Existing Disk

```bash
# Create VM without disk
qm create 100 --memory 4096 --cores 2 --name import-test

# Import disk image
qm importdisk 100 /path/to/image.qcow2 local

# Attach as SCSI
qm set 100 --scsi0 local:vm-100-disk-0

# Or as IDE
qm set 100 --ide0 local:vm-100-disk-0
```

---

## Disk Configuration

### Disk Bus Types

| Bus | Performance | Driver Required | Use Case |
|-----|-------------|-----------------|-----------|
| VirtIO | Best | VirtIO | Linux, modern Windows |
| SCSI | Good | LSI Logic | Legacy, Solaris |
| SATA | Medium | AHCI | Simple VMs |
| IDE | Low | PIIX4 | Legacy only |

### Create Disk

```bash
# VirtIO (recommended)
qm set 100 --scsi0 local:vm-100-disk-0,size=64G,discard=1,ssd=1

# SCSI
qm set 100 --sata0 local:vm-100-disk-1,size=32G

# IDE
qm set 100 --ide0 local:vm-100-disk-2,size=16G
```

### Disk Options

```bash
# Syntax
storage:vm-ID-disk-N,size=SIZE,discard=1,ssd=1,iothread=1,backup=1

# Options:
# - discard: 1 (TRIM support, SSD recommended)
# - ssd: 1 (SSD emulation)
# - iothread: 1 (dedicated I/O thread)
# - backup: 0 (exclude from backup)
# - replicate: 0 (exclude from replication)
# - quota: 50G (NFS Ganesha)
# - werror: stop (write error handling)
```

### Cache Modes

| Mode | Description | Data Safety | Performance |
|------|------------|-------------|------------|
| none | No caching | Unsafe | Best |
| writethrough | Write to cache+disk | Safe | Good |
| writeback | Write to cache | Unsafe | Best |
| directsync | Sync writes | Safe | Slow |
| unsafe | No sync | Unsafe | Best |

```bash
# Set cache mode
qm set 100 --scsi0 local:vm-100-disk-0,cache=writeback
```

### Resize Disk

```bash
# Resize disk
qm resize 100 scsi0 +50G

# Shrink (not recommended)
qm resize 100 scsi0 -10G
```

### Disk Images

```bash
# Create RAW image
qm alloc local 100-disk-0 50G

# Create QCOW2 image
qm alloc local 100-disk-0 50G --format qcow2

# Convert
qemu-img convert -O qcow2 /source.img /dest.qcow2
```

---

## CPU Configuration

### CPU Types

| Type | Description | Compatibility | Migration |
|------|-------------|--------------|-------------|
| host | Pass-through | May fail | No |
| Penryn | Emulated | Good | Yes |
| Conroe | Emulated | Good | Yes |
| Nehalem | Emulated | Good | Yes |
| Westmere | Emulated | Good | Yes |
| SandyBridge | Emulated | Good | Yes |
| IvyBridge | Emulated | Good | Yes |
| Haswell | Emulated | Good | Yes |
| Broadwell | Emulated | Good | Yes |
| Skylake | Emulated | Good | Yes |
| skylake-Client | Emulated | Good | Yes |
| Cascadelake | Emulated | Good | Yes |
| CooperLake | Emulated | Good | Yes |
| IceLake | Emulated | Good | Yes |
| IceLake-Client | Emulated | Good | Yes |
| x86-64v2 | Emulated | Best | Yes |
| x86-64v3 | Emulated | Good | Yes |

### CPU Configuration

```bash
# Set CPU
qm set 100 --cpu host

# Set cores
qm set 100 --cores 4

# Set sockets
qm set 100 --sockets 2

# Set CPU limit (percentage)
qm set 100 --cpulimit 200

# Set CPU units (priority 1024=normal)
qm set 100 --cpuunits 2048

# CPU topology
qm set 100 --cores 4 --sockets 1
```

### NUMA

```bash
# Enable NUMA
qm set 100 --numa 1

# NUMA configuration
qm set 100 --numa 1 --numa0 memory=4096,cpus=0-1
qm set 100 --numa 1 --numa1 memory=4096,cpus=2-3
```

### CPU Flags

```bash
# View available flags
qm cap 100

# Add/remove flags
qm set 100 --cpu "+aes,+pclmuldq" "-avx"
```

---

## Memory Configuration

### Basic Memory

```bash
# Set memory
qm set 100 --memory 8192

# Set balloon (optional)
qm set 100 --balloon 4096

# Balloon enables memory overcommit
# VM can use less than allocated
```

### Huge Pages

```bash
# Enable huge pages
qm set 100 --hugepages 1024

# Configure on host
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

### Memory Options

```bash
# Set memory and swap
qm set 100 --memory 8192 --swap 2048

# Lock memory (prevent swapping)
qm set 100 --locklock 1
```

---

## Network Configuration

### Network Models

| Model | Performance | Driver Required | Use Case |
|-------|-----------|-----------------|----------|
| VirtIO | ~10 Gbps | VirtIO driver | Best |
| vmxnet3 | ~10 Gbps | VMware driver | VMware |
| e1000 | 1 Gbps | Built-in | Legacy |
| rtl8139 | 100 Mbps | Built-in | Old |

### Network Configuration

```bash
# VirtIO (recommended)
qm set 100 --net0 virtio,bridge=vmbr0,macaddr=XX:XX:XX:XX:XX:XX

# e1000
qm set 100 --net0 e1000,bridge=vmbr0,macaddr=XX:XX:XX:XX:XX:XX

# vmxnet3
qm set 100 --net0 vmxnet3,bridge=vmbr0,macaddr=XX:XX:XX:XX:XX:XX
```

### Multi-Queue

```bash
# Enable multi-queue
qm set 100 --net0 virtio,bridge=vmbr0,queues=4
```

### VLAN

```bash
# VLAN tag
qm set 100 --net0 virtio,bridge=vmbr0,vlan=100
```

### Hot-Plug Network

```bash
# Add network device
qm set 100 --net1 virtio,bridge=vmbr1

# Remove network device
qm set 100 --net1 delete
```

---

## GPU Passthrough

### Requirements

1. **CPU with VT-d / AMD-Vi**
2. **Separate GPU** (not for host)
3. **BIOS enabled**: IOMMU
4. **Kernel parameters**: `intel_iommu=on` or `amd_iommu=on`

### Enable IOMMU

```bash
# Edit /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"

# Update GRUB
update-grub

# Reboot
reboot
```

### Find GPU

```bash
# List PCI devices
lspci | grep -i vga

# 01:00.0 VGA compatible controller: NVIDIA Corporation ...
```

### Passthrough

```bash
# Add GPU to VM
qm set 100 --hostpci0 01:00,pcie=1,x-vga=1

# Options:
# - pcie: 1 (PCI Express)
# - x-vga: 1 (primary GPU)
# - romfile: (custom VBIOS)
# - primary: 1 (primary display)
```

### Multiple GPUs

```bash
# Add second GPU
qm set 100 --hostpci1 02:00,pcie=1

# Add video device
qm set 100 --hostpci2 01:00.1,pcie=1
```

### NVIDIA vGPU

```bash
# Check NVIDIA license server
# Requires vGPU license for production
```

### Intel GPU

```bash
# Intel integrated GPU
qm set 100 --hostpci0 00:02.0,pcie=1,x-vga=1
```

### Verify

```bash
# In VM, check
lspci | grep -i vga
# Should show GPU

nvidia-smi  # NVIDIA
```

---

## USB Passthrough

### USB Device

```bash
# Find device
lsusb

# Add USB device
qm set 100 --usb0 host=1050:0407

# USB3
qm set 100 --usb0 xhci,host=1050:0407
```

### USB Controller

```bash
# Add USB controller
qm set 100 --usb0 qemu-xhci,pciex=1

# Or EHCI
qm set 100 --usb0 ich9-ehci1,pciex=1
```

---

## VirtIO Drivers

### Windows

```bash
# Download VirtIO drivers
# https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win-0.1.XX.iso

# Mount to VM
qm set 100 --ide2 local:iso/virtio-win.iso,media=cdrom

# In Windows, install from CD:
# VirtIO driver ISO → Balloon
# VirtIO driver ISO → NetKVM
# VirtIO driver ISO → viostor
# VirtIO driver ISO → vioscsi
# VirtIO driver ISO → qxl
```

### Linux

```bash
# VirtIO drivers usually included
# Install if needed
apt install qemu-guest-agent

# Enable in VM
qm set 100 --agent 1
```

### VirtIO Drivers List

| Driver | Purpose | 
|--------|--------|
| viostor | Block device |
| netkvm | Network |
| vioscsi | SCSI |
| Balloon | Memory balloon |
| qxl | QXL video |
| serine | Serial |
| iga32 | 32-bit graphics |
| pvpanic | Panic device |

---

## Cloud-Init

### Add Cloud-Init

```bash
# Create cloud-init disk
qm set 100 --cicustom "user:local:cloudinit/cloud-init.yml"

# Generate cloud-init
qm cloudinit dump 100

# Cloud-init config example
# cloud-init.yml:
user: admin
name: admin
password: $encrypted_password
ssh-authorized-keys:
  - ssh-rsa AAAAB...
runcmd:
  - echo "hello"
write_files:
  - path: /test
    content: |
      test content
```

### cloud-init with Config Drive

```bash
# Enable config drive
qm set 100 --configdrive 1

# Use for metadata
# /configdrive/
# ├── meta-data
# ├── user-data
# └── vendor-data
```

---

## Operations

### Lifecycle

```bash
# Start
qm start 100

# Stop
qm stop 100

# Restart
qm reset 100

# Pause
qm pause 100

# Resume
qm resume 100

# Suspend
qm suspend 100

# Shutdown (graceful)
qm shutdown 100
```

### Console

```bash
# VNC console
qm vncproxy 100
# Access at: https://host:5900

# CLI console
qm console 100
# Exit: Ctrl+O, Q
```

### Clone

```bash
# Full clone
qm clone 100 101 --full --name "clone"

# Linked clone
qm clone 100 102 --linked --name "linked"

# Clone to different storage
qm clone 100 103 --full --storage local2
```

### Snapshot

```bash
# Create snapshot
qm snapshot 100 pre-update

# List snapshots
qm listsnapshot 100

# Revert
qm rollback 100 pre-update

# Delete
qm delsnapshot 100 pre-update
```

### Migrate

```bash
# Offline migration
qm migrate 100 pve2

# Live migration
qm migrate 100 pve2 --online

# With storage migration
qm migrate 100 pve2 --storage local2
```

### Backup

```bash
# Stop backup
vzdump 100 --mode stop --storage local

# Suspend backup
vzdump 100 --mode suspend --storage local

# Snapshot backup
vzdump 100 --mode snapshot --storage local
```

### Template

```bash
# Convert to template
qm template 100

# Use template
qm clone 100 101
```

---

## Performance Tuning

### VirtIO

```bash
# Enable VirtIO everywhere
qm set 100 --scsi0 local:vm-100-disk-0,discard=1,iothread=1
qm set 100 --net0 virtio,bridge=vmbr0,queues=4
qm set 100 --cpu host
```

### I/O Thread

```bash
# Dedicated I/O thread
qm set 100 --scsi0 local:vm-100-disk-0,iothread=1
```

### Multi-Queue

```bash
# Network multi-queue
qm set 100 --net0 virtio,bridge=vmbr0,queues=4
```

### CPU Pinning

```bash
# Pin to specific cores
qm set 100 --cores 2 --cpuset 0-1
```

### Huge Pages

```bash
# Use huge pages
qm set 100 --hugepages 1024
```

### Disable Features

```bash
# Disable unnecessary
qm set 100 --acpi 0
qm set 100 --tablet 0
qm set 100 --kvm 1
```

---

## Security

### Secure Boot

```bash
# Enable UEFI Secure Boot
qm set 100 --bios ovmf
qm set 100 --efidisk0 local:vm-100-disk-0,efitype=4m,pre-enrolled-keys=1
```

### TPM

```bash
# Add TPM
qm set 100 --tpmstate0 local:vm-100-disk-0,version=2.0
```

### vTPM

```bash
# Software TPM
qm set 100 --tpmstate0 local:vm-100-disk-0,emulator=/usr/share/qemu/swtpm/tpm2-emulator/tpm2-emulator
```

### Disable Features

```bash
# Disable
qm set 100 --disable 1
qm set 100 --lock 1
```

---

## Common Use Cases

### Windows Server

```bash
# Create Windows VM
qm create 200 \
  --name "windows-server" \
  --memory 16384 \
  --cores 4 \
  --cpu host \
  --ostype win10 \
  --scsi0 local:vm-200-disk-0,size=128G \
  --net0 e1000,bridge=vmbr0 \
  --cdrom local:iso/windows-server-2022.iso \
  --bios ovmf
```

### Linux Server

```bash
# Create Linux VM
qm create 201 \
  --name "linux-server" \
  --memory 8192 \
  --cores 4 \
  --cpu host \
  --ostype l26 \
  --scsi0 local:vm-201-disk-0,size=64G,discard=1 \
  --net0 virtio,bridge=vmbr0 \
  --cdrom local:iso/ubuntu-22.04.iso \
  --onboot 1
```

### pfSense Firewall

```bash
# Create pfSense VM
qm create 202 \
  --name "pfsense" \
  --memory 2048 \
  --cores 2 \
  --cpu host \
  --ostype l26 \
  --net0 e1000,bridge=vmbr0 \
  --net1 e1000,bridge=vmbr1 \
  --net2 e1000,bridge=vmbr2 \
  --cdrom local:iso/pfsense.iso
```

### FreeNAS/TrueNAS

```bash
# Create storage VM
qm create 203 \
  --name "truenas" \
  --memory 32768 \
  --cores 4 \
  --cpu host \
  --ostype l26 \
  --scsi0 local:vm-203-disk-0,size=32G \
  --sata1 local:data1,size=1T \
  --sata2 local:data2,size=1T \
  --net0 virtio,bridge=vmbr0
```

---

## Troubleshooting

### No Boot

```bash
# Check boot order
qm show boot 100

# Set boot device
qm set 100 --boot order=scsi0;ide2
```

### Slow Performance

```bash
# Check CPU type
qm config 100 | grep cpu

# Enable host passthrough
qm set 100 --cpu host
```

### Network Issues

```bash
# Check network model
qm config 100 | grep net0

# Change to VirtIO
qm set 100 --net0 virtio,bridge=vmbr0
```

---

## Keywords

#vm #kvm #qemu #virtualization #virtio #gpu-passthrough #pci-passthrough #cloud-init #performance #windows #linux #bios #uefi #secure-boot #tpm

## Links

- [[Proxmox-VE]] - Main documentation
- [[Containers]] - LXC containers
- [[Storage]] - VM storage
- [[Networking]] - Network configuration
- [[Cluster]] - HA VMs
- [[Backup]] - VM backup
- [[Security]] - VM security