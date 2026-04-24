---
title: vGPU Configuration Guide
---

# vGPU Configuration Guide

## Overview

vGPU (virtual GPU) allows partitioning a physical GPU into multiple virtual GPUs that can be assigned to different VMs. Essential for graphics-intensive workloads and Virtual Desktop Infrastructure (VDI).

## Types of vGPU

### NVIDIA vGPU

- **Tesla**: Data center GPUs (T4, A10, A100)
- **Quadro**: Professional GPUs (RTX 6000, 8000)
- **GRID**: Virtualization-specific GPUs

### Intel GVT-g

- **Integrated GPUs**: Intel HD/UHD Graphics
- **Available**: Intel 6th gen+ CPUs

### AMD MxGPU

- **FirePro**: Professional GPUs
- **S7150, S7100**: Series

## Requirements

### Hardware

- GPU with vGPU support
- Enough GPU memory for partitions
- VT-d/IOMMU enabled

### Software

- Proxmox VE 8.0+
- vGPU driver (NVIDIA/Intel/AMD)
- License (NVIDIA GRID)

## NVIDIA vGPU Setup

### Install vGPU Manager

```bash
# Download from NVIDIA portal
wget NVIDIA-vGPU-16-13.8.0-16.13.8.x86_64.run

# Make executable
chmod +x NVIDIA-vGPU-*.run

# Install
./NVIDIA-vGPU-*.run

# Or extract and install
./NVIDIA-vGPU-*.run --extract-only
cd /tmp/NVIDIA-vGPU-*/DriverLink
./nv-carrier --dry-run
./nv-carrier --install

# Reboot
reboot
```

### Verify Installation

```bash
# Check driver
nvidia-smi

# Check vGPU capabilities
ls /sys/class/mdev_bus/

# List available profiles
cat /sys/class/mdev_bus/mdev0/device/mdev_supported_types/*/name
```

### Create vGPU

```bash
# Check available GPUs
lspci | grep -i nvidia

# Create vGPU (example)
echo "nvidia-grid-p40-4q" > /sys/class/mdev_bus/mdev0/device/mdev_supported_types/nvidia-grid-p40-4q/create

# Verify
ls -la /sys/class/mdev_bus/mdev0/

# Get UUID
cat /sys/class/mdev_bus/mdev0/device/mdev_supported_types/nvidia-grid-p40-4q/available_instances
```

### vGPU Profiles

| Profile | GPU Memory | vCPUs | Usability |
|---------|-----------|-------|----------|
| nvidia-grid-p40-4q | 4GB | 4 | Light work |
| nvidia-grid-p40-8q | 8GB | 8 | Workstation |
| nvidia-grid-p40-16q | 16GB | 16 | Heavy load |
| nvidia-grid-p40-24q | 24GB | 24 | Full GPU |

### Assign to VM

```bash
# Get vGPU UUID
ls -la /sys/class/mdev_bus/mdev0/device/

# Assign to VM
qm set 100 --hostpci0 01:00.0,pcie=1,vgpu=uuid

# Using pcie format
qm set 100 --hostpci0 01:00,pcie=1,x-vga=1

# Alternative using mdev
qm set 100 --hostpci0 01:00.0,mdev=nvidia-grid-p40-4q,pcie=1
```

## Intel GVT-g Setup

### Enable GVT-g

```bash
# Add kernel parameter
nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt i915.enable_gvt=1"

# Update GRUB
update-grub

# Reboot
reboot
```

### Check Support

```bash
# Verify GVT-g enabled
dmesg | grep -i gvt

# List available types
ls /sys/class/mdev_bus/*/device/mdev_supported_types/

# Available profiles
find /sys/devices -name i915-*
```

### Create Intel vGPU

```bash
# Check GPU
lspci | grep -i vga

# List available
ls /sys/class/mdev_bus/

# Create vGPU
echo "i915-GVTg_V5_4" > /sys/class/mdev_bus/mdev0/device/mdev_supported_types/i915-GVTg_V5_4/create

# Verify
ls /sys/class/mdev_bus/mdev0/
```

### Intel vGPU Profiles

| Profile | Memory | Use Case |
|---------|--------|----------|
| i915-GVTg_V5_1 | 1GB | Basic |
| i915-GVTg_V5_2 | 2GB | Office |
| i915-GVTg_V5_4 | 4GB | Workstation |
| i915-GVTg_V5_8 | 8GB | Heavy |

### Assign to VM

```bash
# Assign to VM
qm set 100 --hostpci0 00:02.0,pcie=1,mdev=i915-GVTg_V5_4
```

## AMD MxGPU Setup

### Install MxGPU

```bash
# Download from AMD support portal
# Install using amdgpu-pro installer
./amdgpu-pro-install -- virtualization

# Configure
modprobe amdgpu-vgpu

# Reboot
reboot
```

### Create MxGPU

```bash
# Check
ls /sys/class/mdev_bus/

# Create
echo "amd-mxgpu-4" > /sys/class/mdev_bus/mdev0/device/mdev_supported_types/amd-mxgpu-4/create
```

## VM Configuration

### Windows Guest

```bash
# Install NVIDIA/Intel driver in VM
# Download from vendor portal

# For NVIDIA:
# Install GRID driver
# Activate license (automatic or manual)

# Verify
nvidia-smi in VM
```

### Linux Guest

```bash
# Install driver
apt install nvidia-driver-525

# Or for vGPU
apt install nvidia-vgpu-guest

# Verify
nvidia-smi
```

## Performance Tuning

### Memory Allocation

```bash
# Choose appropriate profile based on workload
# - Design/CAD: 8GB+ profiles
# - Office: 2GB profile
# - Light: 1GB profile
```

### Licensing

```bash
# NVIDIA GRID license
# Configure license server
nvfliplib-cli -s license-server

# Manual activation
nvfliplib-cli -a -c "XXXX-XXXX-XXXX"
```

## Use Cases

### VDI (Virtual Desktop)

```bash
# Profile for office workers
# i915-GVTg_V5_2 or nvidia-grid-p40-4q

# Create pool
for i in 200..210; do
  qm create $i --name "vdi-$i" \
    --memory 4096 --cores 2 \
    --hostpci0 00:02.0,mdev=i915-GVTg_V5_2
done
```

### CAD/Design

```bash
# High-performance profile
# nvidia-grid-p40-16q or nvidia-grid-p40-24q

qm create 300 --name "cad-workstation" \
  --memory 32768 --cores 8 \
  --hostpci0 01:00,pcie=1,x-vga=1,mdev=nvidia-grid-P40-16Q
```

### Machine Learning

```bash
# Large GPU allocation
qm create 400 --name "ml-workstation" \
  --memory 65536 --cores 16 \
  --hostpci0 01:00,pcie=1,mdev=nvidia-grid-P40-24Q
```

## Troubleshooting

### vGPU Not Available

```bash
# Check driver
lsmod | grep -E "kvm|nvidia"

# Check IOMMU
dmesg | grep -i iommu

# Check license
nvidia-smi -q | grep -i license
```

### Performance Issues

```bash
# Monitor GPU usage
nvidia-smi

# Check for throttling
nvidia-smi -q

# Profile memory
nvidia-smi -q -d MEMORY
```

## Keywords

#vgpu #nvidia #intel-gvt #amd-mxgpu #gpu-virtualization #vdi #desktop-virtualization #grid

## Related Articles

- [[Advanced]]
- [[PCI-Passthrough]]
- [[SR-IOV]]
- [[Performance]]
---

[[index|Back to Proxmox VE]]
