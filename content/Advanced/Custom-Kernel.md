# Custom Kernel Configuration

## Overview

Compile and configure custom kernels for specific hardware support, performance tuning, or security requirements.

## Build Custom Kernel

### Install Build Dependencies

```bash
# Install build tools
apt install build-essential git

# Install kernel build deps
apt build-dep linux
```

### Clone Kernel Source

```bash
# Clone Proxmox kernel
git clone https://github.com/proxmox/linux-proxmox.git

# Or use vanilla kernel
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
```

### Configure Kernel

```bash
# Change to source directory
cd linux-proxmox

# Use Proxmox config
cp /boot/config-$(uname -r) .config

# Or use menuconfig
make menuconfig
```

### Build Kernel

```bash
# Build
make -j$(nproc) deb-pkg

# Install
dpkg -i ../linux-*.deb
```

## Kernel Parameters

### GRUB Configuration

```bash
# Edit GRUB
nano /etc/default/grub

# Add parameters
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

# Update GRUB
update-grub
```

### Common Parameters

| Parameter | Description |
|-----------|-------------|
| intel_iommu=on | Enable IOMMU |
| amd_iommu=on | Enable AMD IOMMU |
| iommu=pt | IOMMU passthrough |
| hugepages=1024 | Huge pages |
| default_hugepagesz=2M | 2MB pages |
| transparent_hugepage=never | Disable THP |
| nosmt | Disable SMT |

## Kernel Modules

### Load at Boot

```bash
# Create config
nano /etc/modules

# Add modules
vfio
vfio_iommu_type1
vfio_pci
nvidia
nvidia_uvm
```

### Blacklist Modules

```bash
# Blacklist
nano /etc/modprobe.d/blacklist.conf

blacklist nouveau
blacklist nvidia
```

## Performance Tuning

### CPU Governors

```bash
# Set governor
echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Make persistent
apt install cpufrequtils
nano /etc/default/cpufrequtils
```

### Kernel Tuning

```bash
# Network buffer
net.core.rmem_max=16777216
net.core.wmem_max=16777216

# File system
fs.file-max=65536
```

## Security Hardening

### Kernel Settings

```bash
# Disable unused features
echo 1 > /proc/sys/kernel/dmesg_restrict

# Disable core dumps
hard core 0

# Restrict sysrq
echo 1 > /proc/sys/kernel/sysrq
```

## Keywords

#kernel #compilation #tuning #performance #security #kernel-parameters

## Related Articles

- [[Advanced]]
- [[Performance-Tuning]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
