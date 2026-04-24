# PCI Passthrough Complete Guide

## Overview

PCI Passthrough allows direct assignment of physical PCIe devices to virtual machines, providing near-native performance for hardware-dependent workloads.

## Why Use PCI Passthrough

### Benefits

- Near-native performance (95%+)
- Direct hardware access
- No virtualization overhead
- Full feature utilization

### Use Cases

- GPU Computing (CUDA, OpenCL)
- High-performance networking
- Hardware security modules
- Audio/Video capture
- Dongle-based licensing

## Requirements

### Hardware

- CPU with VT-d (Intel) or AMD-Vi (AMD)
- Motherboard with IOMMU support
- Compatible PCIe device

### Software

- Proxmox VE 8.0+
- IOMMU enabled in kernel

## Enable IOMMU

### Check Current Status

```bash
# Check CPU support
egrep -c 'vmx|svm' /proc/cpuinfo

# Check IOMMU groups
find /sys/kernel/iommu_groups/ -type l | head -20
```

### Configure GRUB

```bash
# Edit GRUB
nano /etc/default/grub

# For Intel:
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"

# For AMD:
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"

# Update GRUB
update-grub

# Reboot
reboot
```

### Verify IOMMU

```bash
# Check kernel messages
dmesg | grep -e DMAR -e IOMMU

# List IOMMU groups
find /sys/kernel/iommu_groups/ -type l | wc -l
```

## GPU Passthrough

### Identify GPU

```bash
# List GPUs
lspci -nn | grep -E "VGA|3D"

# Example output:
# 01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP104 [10de:1b80]
# 01:00.1 Audio device [0403]: NVIDIA Corporation GP104 HD Audio [10de:0fbb]
```

### Enable Primary GPU

```bash
# Add to VM
qm set 100 --hostpci0 01:00,pcie=1,x-vga=1

# Options explanation:
# --hostpci0: First PCI device
# 01:00: PCI address
# pcie=1: PCIe mode
# x-vga=1: VGA primary
```

### Multiple GPUs

```bash
# Add second GPU
qm set 100 --hostpci1 02:00,pcie=1

# Add audio (for GPU)
qm set 100 --hostpci0 01:00.1,pcie=1
```

### GPU Options

```bash
# Full configuration
qm set 100 --hostpci0 01:00,pcie=1,x-vga=1,romfile=/var/lib/vz/template/efi-nvidia.bin

# Explanation:
# pcie=1: PCIe mode (required for modern GPUs)
# x-vga=1: Use as primary display
# romfile: Custom VBIOS (optional)
```

## Network Card Passthrough

### Identify Network Card

```bash
# List network devices
lspci -nn | grep -E "Ethernet|Network"

# Example:
# 02:00.0 Ethernet controller [0200]: Intel Corporation I350 [8086:1521]
```

### Passthrough Network Card

```bash
# Add to VM
qm set 100 --hostpci0 02:00,pcie=1

# Or use directly as network device
qm set 100 --net0 virtio,bridge=vmbr0
```

### SR-IOV Network Cards

```bash
# Create VFs first (see SR-IOV article)
echo 2 > /sys/class/net/eth0/device/sriov_numvfs

# Passthrough VF
qm set 100 --hostpci0 01:00.2,pcie=1
```

## USB Controller Passthrough

### Identify USB Controller

```bash
# List USB controllers
lspci -nn | grep -E "USB|xHCI"

# Example:
# 00:14.0 USB controller [0c03]: Intel Corporation xHCI [8086:a12f]
```

### Passthrough USB Controller

```bash
# Add to VM
qm set 100 --hostpci0 00:14,pcie=1

# Add multiple
qm set 100 --hostpci0 00:14,pcie=1 --hostpci1 00:1a,pcie=1
```

### USB Device Passthrough

```bash
# List USB devices
lsusb

# Add specific device (USB redirect)
qm set 100 --usb0 host=046d:c52b

# USB 3.0
qm set 100 --usb0 xhci,pcie=1
```

## HBA Passthrough

### Identify HBA

```bash
# List SAS/SATA controllers
lspci -nn | grep -E "SAS|SATA|RAID"

# Example:
# 03:00.0 RAID controller [0104]: LSI Logic SAS-3 3008 [1000:0012]
```

### Passthrough HBA

```bash
# Add to VM
qm set 100 --hostpci0 03:00,pcie=1

# Use as boot device
qm set 100 --boot order=scsi0
```

## NVMe Passthrough

### Identify NVMe

```bash
# List NVMe devices
lspci -nn | grep -E "Non-Volatile"

# Example:
# 04:00.0 Non-Volatile memory controller [0108]: Samsung Electronics NVMe [144d:a802]
```

### Passthrough NVMe

```bash
# Add to VM
qm set 100 --hostpci0 04:00,pcie=1

# Configure as boot
qm set 100 --boot order=scsi0
```

## Security Considerations

### Device Isolation

```bash
# Check IOMMU group
readlink /sys/bus/pci/devices/0000:01:00.0/iommu_group

# Must be in separate group for passthrough
# Same group = cannot passthrough individually
```

### Alternative: vfio-pci

```bash
# Bind device to vfio-pci
echo 0000:01:00.0 > /sys/bus/pci/drivers/vfio-pci/new_id

# Check
lspci -nnk -d 10de:
```

## Troubleshooting

### Device Not Available

```bash
# Check device in IOMMU group
find /sys/kernel/iommu_groups/ -type l -name "*01:00*"

# Must be separate group
# If not, enable ACS override
echo "options pci_ids default_hugepagesz=2M" >> /etc/modprobe.d/kvm.conf
```

### VM Won't Start

```bash
# Check device status
lspci -nnk -s 01:00

# Check it's not in use
lsof | grep 01:00

# Check device available
qm list
```

### Performance Issues

```bash
# Enable IOAPIC
qm set 100 --cpu host

# Use PCIe
qm set 100 --hostpci0 01:00,pcie=1
```

## Windows VM Configuration

### Install Drivers

```bash
# After passthrough, install drivers in Windows
# Device Manager shows Unknown Device

# Install:
# - GPU: NVIDIA/AMD driver
# - Network: Intel driver
# - USB: Native
```

### GPU Performance

```bash
# Enable optimized settings
# NVIDIA Control Panel -> Manage 3D Settings
# AMD Settings -> Gaming
```

## Linux VM Configuration

### Install Drivers

```bash
# GPU
apt install nvidia-driver-525

# Network
apt install firmware-linux

# Check
lspci -nnk
```

## Best Practices

1. **Check IOMMU**: Verify separate groups
2. **Use PCIe**: Modern VMs
3. **Device cleanup**: Remove on VM delete
4. **Backup ROM**: Save VBIOS
5. **Monitor temps**: GPU throttling

## Keywords

#pci-passthrough #iommu #vfio #gpu-passthrough #usb-passthrough #network-passthrough #vt-d #amd-vi

## Related Articles

- [[Advanced]]
- [[SR-IOV]]
- [[vGPU-Configuration]]
- [[Nested-Virtualization]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
