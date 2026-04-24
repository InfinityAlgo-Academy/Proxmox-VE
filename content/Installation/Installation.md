---
title: Proxmox VE Installation
---

# Proxmox VE Installation

## Installation Methods

### 1. ISO Installation (Recommended)

```bash
# Download Proxmox VE ISO
wget https://www.proxmox.com/images/proxmox/release/ProxmoxVE_8.0-iso.tar.zsync
# Or use the direct ISO
# Create bootable USB
sudo dd if=proxmox-8.0.iso of=/dev/sdX bs=1M status=progress
```

### 2. PXE Network Boot

```bash
# Configure DHCP for PXE
next-server 192.168.1.100;
filename "pxelinux.0";
```

## Installation Steps

1. **Boot from USB/ISO**
2. **Select "Install Proxmox VE"**
3. **Choose Target Disk** (or configure RAID first)
4. **Set Location/Timezone**
5. **Set Admin Password** + Email
6. **Network Configuration**:
   - Management IP: `192.168.1.x/24`
   - Gateway: `192.168.1.1`
   - Hostname: `pve1.yourdomain.com`
7. **Review & Install**

## Post-Installation

### Update Proxmox
```bash
apt update && apt full-upgrade -y
# Reboot after kernel update
reboot
```

### Configure Repository
```bash
# Disable Enterprise repo (for unpaid)
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list
# Enable community repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-community.list
```

### Access Web Interface
- URL: `https://192.168.1.100:8006`
- User: `root`
- Password: (set during install)

## Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4GB | 8GB+ |
| System Disk | 32GB | 64GB+ SSD |
| VM Storage | 50GB | 500GB+ |

## Advanced Installation

### On Existing Debian

```bash
# Add Proxmox repository
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
wget https://enterprise.proxmox.com/debian/proxmox-release-8.gpg -O /etc/apt/trusted.gpg.d/proxmox-release.asc

# Install Proxmox
apt update && apt install proxmox-ve postfix open-iscsi -y

# Remove old virtualization
apt remove qemu-kvm -y
apt autoremove -y

# Reboot
reboot
```

### ZFS Root Installation

During installation, select "ZFS (Mirror)" for:
- Redundancy
- Copy-on-write
- Snapshots
- Compression

```bash
# ZFS commands after install
zpool status
zfs list
# Create snapshot
zfs snap poolname/rpool@backup-2024-01-01
```

## Troubleshooting

See [[Troubleshooting]] for common installation issues.

## Keywords

#proxmox #installation #setup #debian #zfs #raid #iso #usb #pxe

## Links

- [[Proxmox-VE]] - Main documentation
- [[Storage]] - Disk configuration
- [[Networking]] - Network setup
- [[Troubleshooting]] - Installation issues
---

[[index|Back to Proxmox VE]]
