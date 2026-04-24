# Proxmox VE Hardware Guide

## Table of Contents

1. [Hardware Requirements](#hardware-requirements)
2. [CPU Recommendations](#cpu-recommendations)
3. [Memory Guide](#memory-guide)
4. [Storage Hardware](#storage-hardware)
5. [Network Hardware](#network-hardware)
6. [Server Recommendations](#server-recommendations)
7. [GPU Passthrough](#gpu-passthrough)
8. [USB Passthrough](#usb-passthrough)
9. [Hardware Compatibility](#hardware-compatibility)
10. [Home Lab Builds](#home-lab-builds)

---

## Hardware Requirements

### Minimum Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 2GB | 4GB |
| System Disk | 8GB | 32GB+ SSD |
| Network | 1GbE | 1GbE |
| CPU Support | VT-x/AMD-V | VT-d/AMD-Vi |

### Virtualization Requirements

```bash
# Check CPU virtualization
egrep -c '(vmx|svm)' /proc/cpuinfo
# Should return > 0

# Check VT-d (Intel)
dmesg | grep -E "(DMAR|IOMMU)"

# Check AMD-Vi
dmesg | grep AMD-Vi
```

---

## CPU Recommendations

### Home Lab

| CPU | Cores | VMs | Price |
|-----|-------|-----|-------|
| Intel i3-6100 | 4 | 5-10 | $50-80 |
| Intel i5-7500 | 4 | 10-15 | $80-120 |
| Intel i7-7700 | 8 | 15-20 | $120-180 |
| AMD Ryzen 3 2200G | 4 | 8-12 | $60-90 |
| AMD Ryzen 5 2600 | 6 | 15-20 | $80-120 |

### Small Business

| CPU | Cores | VMs | Price |
|-----|-------|-----|-------|
| Intel i9-9900 | 8 | 20-30 | $200-300 |
| AMD Ryzen 7 3700X | 8 | 20-30 | $180-250 |
| Intel Xeon E-2288G | 8 | 25-35 | $300-400 |
| AMD EPYC 7302 | 16 | 40-50 | $500-700 |

### Enterprise

| CPU | Cores | VMs | Price |
|-----|-------|-----|-------|
| Intel Xeon Gold 6248 | 20 | 50-60 | $800-1000 |
| Intel Xeon Platinum 8256 | 32 | 60-80 | $1500-2000 |
| AMD EPYC 7742 | 64 | 80-100 | $2000-3000 |

### CPU Comparison

| Feature | Intel | AMD |
|---------|-------|-----|
| Price | Higher | Lower |
| Performance | Similar | Similar |
| vGPU Support | Better | Limited |
| Power | Higher | Lower |
| ECC Support | Some | All |

---

## Memory Guide

### RAM Requirements

| VM Type | Minimum | Recommended |
|--------|---------|-------------|
| pfSense | 512MB | 2GB |
| Windows | 2GB | 4GB+ |
| Linux | 512MB | 2GB |
| Docker Host | 4GB | 8GB+ |
| Database | 2GB | 8GB+ |

### Memory Allocation Example

```
Total RAM: 32GB

System: 2GB
Reserved: 2GB
Available: 28GB

VMs (6GB each):
  pfSense: 2GB
  Windows: 4GB
  Ubuntu: 2GB

Containers (1GB each):
  16 × 1GB = 16GB
```

### RAM Types

| Type | Speed | ECC | Price |
|------|-------|-----|------|
| DDR4-2666 | 2666MHz | No | $ |
| DDR4-3200 | 3200MHz | No | $$ |
| DDR4-3200 ECC | 3200MHz | Yes | $$$ |
| DDR5-4800 | 4800MHz | Some | $$$$ |

### RAM Installation

```bash
# Check RAM
dmidecode -t memory | grep -A 3 "Size:"
free -h

# Memory info
cat /proc/meminfo
```

---

## Storage Hardware

### Boot Disk

| Type | Size | Performance | Price |
|------|------|-------------|-------|
| SSD SATA | 64GB+ | Good | $ |
| NVMe | 64GB+ | Excellent | $$ |

### VM Storage

| Type | Performance | RAID | Price |
|----------|------------|------|-------|
| HDD | Poor | Not recommended | $ |
| SSD SATA | Good | RAID 1/10 | $$ |
| NVMe | Excellent | RAID 1/10 | $$$ |
| NVMe + HDD Cache | Excellent | Mixed | $$ |

### Storage Configuration

```bash
# Recommended setup:
# Boot: 2x 64GB SSD (RAID 1 mirror)
# VM: 2x 512GB NVMe (RAID 1 mirror)
# Data: Multiple HDD/NVMe as needed
```

### Disk Benchmarks

```bash
# Sequential read
hdparm -t /dev/sda

# Random read
fio --name=randread --ioengine=libaio --bs=4k --direct=1 --size=1G --numjobs=4 --runtime=30

# Check disk type
lsblk -o NAME,ROTA
# ROTA=0: SSD
# ROTA=1: HDD
```

---

## Network Hardware

### Network Cards

| Type | Speed | Use Case | Price |
|------|-------|----------|-------|
| i211 | 1GbE | Basic | $ |
| i310 | 1GbE | Basic | $ |
| i350-T2 | 2x 1GbE | Trunk/Bond | $$ |
| X520 | 2x 10GbE | Storage | $$$ |
| X540 | 2x 10GbE | Storage | $$$ |
| AQN | 10GbE | Storage | $$$ |

### Recommended NICs

| Use Case | Recommended |
|---------|------------|
| Home Lab | Onboard 1GbE |
| Multiple VMs | Intel i350-T2 |
| NFS/iSCSI | Mellanox/Intel X520 |
| vGPU | Any 10GbE |

### Network Commands

```bash
# Check network
lspci | grep -i ethernet

# Check speed
ethtool eth0

# Check driver
ethtool -i eth0

# Check interrupts
cat /proc/interrupts | grep eth0
```

---

## Server Recommendations

### Budget Build ($200-400)

| Component | Recommendation | Price |
|-----------|-------------|-------|
| CPU | Intel i5-7500 | $100 |
| RAM | 16GB (2x8GB) | $60 |
| Boot | 64GB SSD | $30 |
| Storage | 500GB SSD | $50 |
| Case | Fractal Core | $50 |

### Mid-Range Build ($400-800)

| Component | Recommendation | Price |
|-----------|-------------|-------|
| CPU | Intel i9-9900 | $250 |
| RAM | 32GB (4x8GB) | $120 |
| Boot | 128GB NVMe | $40 |
| VM Storage | 1TB NVMe | $100 |
| Network | Intel i350-T2 | $80 |

### High-End Build ($800-1500)

| Component | Recommendation | Price |
|-----------|-------------|-------|
| CPU | Intel Xeon E-2288G | $500 |
| RAM | 64GB ECC | $250 |
| Boot | 2x 128GB NVMe (RAID 1) | $100 |
| VM Storage | 2x 1TB NVMe (RAID 1) | $200 |
| Network | Intel X520 | $150 |

### Enterprise Build ($2000+)

| Component | Recommendation | Price |
|-----------|-------------|-------|
| CPU | 2x Intel Xeon Gold | $2000 |
| RAM | 128GB+ ECC | $800 |
| Boot | 2x 256GB NVMe (RAID 1) | $200 |
| Storage | 4x 2TB NVMe (RAID 10) | $800 |
| Network | 2x 10GbE | $300 |

---

## Server Hardware Recommendations

### Dell OptiPlex

| Model | CPU | RAM Slots | Bays | Notes |
|-------|-----|----------|------|-------|
| 7080 MFF | i7-10700 | 2 | 1x m.2 | Compact |
| 7080 SFF | i7-10700 | 4 | 2x 3.5" | Good choice |
| 7080 MT | i7-10700 | 4 | 2x 3.5" | Best for lab |
| 9020 MT | i7-3770 | 4 | 2x 3.5" | Budget |

### HP ProDesk

| Model | CPU | RAM Slots | Bays | Notes |
|-------|-----|----------|------|-------|
| 600 G6 SFF | i7-9700 | 4 | 2x 3.5" | Good |
| 600 G6 MT | i7-9700 | 4 | 2x 3.5" | Best |
| 600 G5 SFF | i7-9700 | 4 | 2x 3.5" | Budget |

### HP ProLiant

| Model | CPU | RAM Slots | Bays | Notes |
|-------|-----|----------|------|-------|
| DL360 Gen10 | 2x Xeon | 24 | 8x SFF | Enterprise |
| DL380 Gen10 | 2x Xeon | 24 | 24x SFF | Storage |
| DL20 Gen10 | Xeon E | 4 | 2x LFF | Budget |

### Dell PowerEdge

| Model | CPU | RAM Slots | Bays | Notes |
|-------|-----|----------|------|-------|
| R640 | 2x Xeon | 24 | 8x NVMe | Enterprise |
| R740 | 2x Xeon | 24 | 24x SFF | Storage |
| R730 | 2x Xeon | 24 | 12x LFF | Budget |

---

## GPU Passthrough

### Compatible GPUs

| GPU | VFIO Support | Use Case | Price |
|-----|-------------|----------|-------|
| GTX 1060 | Yes | Gaming | $150 |
| GTX 1070 | Yes | Gaming/Transcode | $200 |
| GTX 1080 | Yes | Gaming/Transcode | $250 |
| RTX 3060 | Yes | Gaming/AI | $300 |
| RTX 3080 | Yes | Gaming/AI | $500 |
| Quadro P4000 | Yes | Workstation | $400 |
| Quadro RTX 4000 | Yes | Workstation | $900 |
| Intel iGPU | Yes | Transcode | Included |

### Enable GPU Passthrough

```bash
# 1. Enable IOMMU in BIOS
# Intel: VT-d
# AMD: AMD-Vi

# 2. Add kernel parameters
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"

update-grub
reboot

# 3. Find GPU
lspci | grep -i vga

# 4. Add to VM
qm set VMID --hostpci0 01:00,pcie=1,x-vga=1
```

### NVIDIA vGPU (Enterprise)

```bash
# Requires NVIDIA license
# Use GRID or vGPU software
# Contact NVIDIA for licensing
```

---

## USB Passthrough

### Common USB Devices

| Device | Use Case | Notes |
|--------|----------|-------|
| Z-Wave | Home Automation | Zigbee controller |
| Serial | Hardware | Console |
| Dongle | 2FA | Security key |
| Flash | Boot | Installation |

### USB Passthrough

```bash
# Find USB device
lsusb

# Add to VM
qm set VMID --usb0 host=1050:0407

# Add to LXC
pct set CTID --usb0 host=1050:0407

# Or mount in LXC
pct set CTID --mapping usb,host=1050:0407
```

---

## Hardware Compatibility

### Compatible Hardware

### Network Cards

| Card | Driver | Support |
|------|--------|---------|
| Intel i211 | igb | Excellent |
| Intel i350 | igb | Excellent |
| Intel X520 | ixgbe | Excellent |
| Intel X540 | ixgbe | Excellent |
| Realtek 8111 | r8169 | Good |
| Realtek 8125 | r8125 | Good |
| Mellanox ConnectX | mlx4_core | Excellent |

### HBAs

| Card | Type | Driver |
|------|------|--------|
| LSI 9211-8i | 6Gb SAS | mpt2sas |
| LSI 9300-8i | 12Gb SAS | mpt3sas |
| HBA 12Gb | 12Gb SAS | mpt3sas |
| LSI 9260-8i | 6Gb SAS | mpt2sas |

### USB WiFi (Not Recommended)

```bash
# WiFi not recommended for Proxmox
# Use wired networking
```

---

## Home Lab Builds

### Basic Home Lab ($150)

| Component | Details |
|-----------|----------|
| CPU | Intel i5-6600 |
| RAM | 16GB |
| Boot | 60GB SSD |
| Storage | 500GB HDD |

### Media Server ($300)

| Component | Details |
|-----------|----------|
| CPU | Intel i7-7700 |
| RAM | 32GB |
| Boot | 128GB NVMe |
| Storage | 4TB HDD |

### High-Performance ($600)

| Component | Details |
|-----------|----------|
| CPU | Intel i9-9900K |
| RAM | 64GB |
| Boot | 256GB NVMe |
| Storage | 2TB NVMe + 8TB HDD |
| GPU | GTX 1660 |

---

## Quick Reference

### Hardware Checklist

- [ ] CPU with VT-x/VT-d or AMD-V
- [ ] 4GB+ RAM
- [ ] SSD for system
- [ ] NIC with 1GbE
- [ ] Reliable power supply

### Recommended Brands

| Component | Brands |
|-----------|--------|
| CPU | Intel, AMD |
| RAM | Crucial, Kingston, Samsung |
| SSD | Samsung, WD, Crucial |
| NIC | Intel |
| Case | Fractal, Corsair |

## Keywords

#hardware #cpu #ram #storage #network #gpu-passthrough #usb-passthrough #server #build

## Links

- [[Proxmox-VE]] - Main documentation
- [[Installation]] - Installation guide
- [[VM]] - VM setup
- [[Storage]] - Storage setup
- [[Networking]] - Network setup