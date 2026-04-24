# SR-IOV Configuration Guide

## Overview

Single Root I/O Virtualization (SR-IOV) allows a single PCIe device to appear as multiple separate virtual functions (VFs). This provides near-native performance for virtual machines.

## What is SR-IOV?

### How It Works

- Physical Function (PF): Original PCIe device
- Virtual Functions (VFs): Virtual instances of the device
- Each VF operates independently
- Direct assignment to VMs

### Benefits

- Near-native performance
- Reduced CPU overhead
- Lower latency
- Better throughput

## Requirements

### Hardware Requirements

- CPU with VT-d (Intel) or AMD-Vi (AMD)
- Motherboard with IOMMU support
- Compatible PCIe device

### Software Requirements

- Proxmox VE 8.0+
- IOMMU enabled in BIOS
- Kernel with SR-IOV support

## Enable IOMMU

### Check IOMMU Support

```bash
# Intel
dmesg | grep -e DMAR -e IOMMU

# AMD
dmesg | grep -e AMD-Vi -e IOMMU

# List enabled IOMMU groups
find /sys/kernel/iommu_groups/ -type l
```

### Enable in GRUB

```bash
# Edit GRUB configuration
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

## Network SR-IOV

### Enable on Network Card

```bash
# Check if supported
lspci -vv | grep -i "SR-IOV"

# Example output:
# 0000:01:00.0 Ethernet controller: Intel I350
#    Capabilities: [190] #1 Signal Integrity Information
#    ...
#    Function Level Reset: 0x01
#    TLP Processing Hint: 0x00
#    Max_Num_VFs: 7
```

### Create Virtual Functions

```bash
# For Intel i350
echo 2 > /sys/class/net/eth0/device/sriov_numvfs

# Verify
ip link show eth0
# Output shows VF devices

# Check VF details
lspci | grep -i "Virtual"

# Make persistent
echo 'echo 2 > /sys/class/net/eth0/device/sriov_numvfs' >> /etc/rc.local
```

### Configure VF in VM

```bash
# Assign VF to VM
qm set 100 --net0 virtio,bridge=vmbr0,macaddr=XX:XX:XX:XX:XX:XX

# Find VF address
lspci | grep "Ethernet"
# 0000:01:00.2 Ethernet controller: Intel I350

# Assign to VM
qm set 100 --hostpci0 01:00.2,pcie=1
```

## GPU SR-IOV

### NVIDIA vGPU Requirements

- NVIDIA GPU with vGPU support ( Tesla, Quadro)
- NVIDIA vGPU Manager
- NVIDIA license server

### NVIDIA vGPU Setup

```bash
# Download vGPU Manager from NVIDIA portal
wget NVIDIA-vGPU-16-13.8.0-16.13.8.x86_64.run

# Install
chmod +x NVIDIA-vGPU-*.run
./NVIDIA-vGPU-*.run

# Reboot
reboot

# Verify installation
nvidia-smi
# Shows vGPU instances available
```

### Configure vGPU in VM

```bash
# Check available vGPU profiles
ls /sys/class/mdev_bus/

# Create vGPU
echo "nvidia-grid-p40-4q" > /sys/class/mdev_bus/mdev0/device/mdev_supported_types/nvidia-grid-p40-4q/create

# Assign to VM
qm set 100 --hostpci0 00:0a.0,pcie=1,vgpu=1,uuid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Intel GVT-g

```bash
# Enable Intel GVT-g
echo "i915.enable_gvt=1" > /etc/modprobe.d/gvt.conf

# Update GRUB
update-grub
reboot

# Check
ls /sys/class/mdev_bus/
# Available GPU profiles
```

## Storage SR-IOV

### Enable on HBA

```bash
# Check for SR-IOV capable HBA
lspci | grep -i sas

# Enable VFs
echo 4 > /sys/class/scsi_host/host0/device/sriov_numvfs

# Verify
lspci | grep -i "Serial"
```

## Advanced Configuration

### Virtual Function Management

```bash
# List all VFs
lspci | grep -i VF

# View VF details
lspci -vvv -s 0000:01:00.2

# Configure VF MAC address
ip link set eth0 vf 0 mac AA:BB:CC:DD:EE:FF

# Configure VF VLAN
ip link set eth0 vf 0 vlan 100

# Verify VF statistics
ip -s link show eth0
```

### Persist Configuration

```bash
# Create udev rules
nano /etc/udev/rules.d/99-sriov.rules

# Set MAC for VF0
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="AA:BB:CC:DD:EE:00", ATTR{type}=="1", KERNEL=="eth0", NAME="vf0p0"

# Reload udev
udevadm control --reload-rules
```

## Performance Comparison

### Network Performance

| Configuration | Latency | Throughput |
|---------------|--------|-----------|
| VirtIO | 50μs | 10 Gbps |
| SR-IOV VF | 20μs | 10 Gbps |
| PCI Passthrough | 15μs | 10 Gbps |

### CPU Usage

| Configuration | CPU Overhead |
|---------------|------------|
| VirtIO | 10-15% |
| SR-IOV | 2-5% |
| PCI Passthrough | 1-3% |

## Troubleshooting

### SR-IOV Not Working

```bash
# Check IOMMU
dmesg | grep -i iommu

# Verify BIOS settings
# Enable: Intel VT-d / AMD-Vi
# Enable: Above 4G Decoding (if needed)

# Check kernel
dmesg | grep -i sriov

# Check device
lspci -vvv 0000:01:00 | grep -i sriov
```

### VF Not Available

```bash
# Check MAX VFs
cat /sys/class/net/eth0/device/sriov_totalvfs
cat /sys/class/net/eth0/device/sriov_numvfs

# Enable more VFs
echo 4 > /sys/class/net/eth0/device/sriov_numvfs

# Check driver
ethtool -i eth0
```

## Best Practices

1. **BIOS first**: Enable IOMMU in BIOS
2. **Right card**: Use certified SR-IOV cards
3. **Minimize VFs**: Create only what's needed
4. **Dedicate PFs**: Don't use PF for host
5. **Monitor**: Watch for errors

## Keywords

#sriov #virtual-function #iommu #pci-passthrough #gpu-virtualization #nvidia #intel-gvt #performance

## Related Articles

- [[Advanced]]
- [[PCI-Passthrough]]
- [[vGPU-Configuration]]
- [[Performance]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
