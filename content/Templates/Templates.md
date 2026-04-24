# Proxmox VE Templates Guide

## Table of Contents

1. [LXC Templates](#lxc-templates)
2. [VM Templates](#vm-templates)
3. [Cloud-Init](#cloud-init)
4. [Container OS Templates](#container-os-templates)
5. [Custom Templates](#custom-templates)
6. [Template Variables](#template-variables)
7. [Autoinstall](#autoinstall)
8. [Windows Templates](#windows-templates)

---

## LXC Templates

### Official Templates

```bash
# Update template list
pveam update

# List available
pveam list standard
pveam list community
```

### Download Templates

```bash
# Debian
pveam download debian-12 standard 12
pveam download debian-11 standard 11

# Ubuntu
pveam download ubuntu-22.04 standard 22.04
pveam download ubuntu-20.04 standard 20.04

# CentOS
pveam download centos-7 standard 7
pveam download centos-8 standard 8

# Alpine
pveam download alpine-3.18 standard 3.18
pveam download alpine-3.17 standard 3.17

# Arch
pveam download archlinux current
```

### Template Locations

```bash
# Default location
/var/lib/vz/template/cache/

# Custom location
/mnt/storage/templates/
```

---

## VM Templates

### Creating VM Templates

```bash
# Create VM
qm create 9000 --name "template-ubuntu" --memory 4096 --cores 4

# Install OS and configure
# Configure for templating:
# - Set cloud-init user
# - Install qemu-guest-agent
# - Clean up

# Convert to template
qm template 9000

# Clone from template
qm clone 9000 100 --full --name "new-vm"
```

### Cloud-Init Templates

```bash
# Add cloud-init
qm set 100 --ide2 local:cloudinit

# Generate cloud-init config
qm cloudinit dump 100

# Create user data
cat > /tmp/user-data.yaml << EOF
#cloud-config
users:
  - name: admin
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa PUBLICKEY...
chpasswd:
  list: |
    admin:password
EOF
```

---

## Cloud-Init

### Cloud-Init Configuration

```bash
# Meta Data
instance-id: vm-100
local-hostname: myvm

# User Data
#cloud-config
users:
  - name: admin
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
chpasswd:
  list: |
    admin:password
runcmd:
  - echo "nameserver 1.1.1.1" > /etc/resolv.conf
  - apt update && apt upgrade -y
```

### Cloud-Init Options

```bash
# Set as VM
qm set VMID --ciuser admin
qm set VMID --cipassword password
qm set VMID --cipass1 password
qm set VMID --cienv VAR1=VALUE1
qm set VMID --cienv VAR2=VALUE2
```

### Network Config (Cloud-Init)

```yaml
# network-config
version: 2
ethernets:
  eth0:
    addresses:
      - 192.168.1.100/24
    gateway4: 192.168.1.1
    nameservers:
      addresses:
        - 1.1.1.1
        - 8.8.8.8
```

---

## Container OS Templates

### Debian

```bash
# Debian 12 (Bookworm)
pveam download debian-12 standard 12

# Debian 11 (Bullseye)
pveam download debian-11 standard 11

# Configure after creation
pct exec CTID -- apt update && apt upgrade -y
pct exec CTID -- apt install -y htop curl wget git
```

### Ubuntu

```bash
# Ubuntu 22.04 LTS
pveam download ubuntu-22.04 standard 22.04

# Ubuntu 20.04 LTS
pveam download ubuntu-20.04 standard 20.04

# Post-install
pct exec CTID -- apt update
pct exec CTID -- apt upgrade -y
pct exec CTID -- locale-gen en_US.UTF-8
```

### Alpine

```bash
# Alpine Linux
pveam download alpine-3.18 standard 3.18

# Quick setup
pct exec CTID -- setup-interfaces -a
pct exec CTID -- rc-update add networking
```

### CentOS/Rocky/AlmaLinux

```bash
# Rocky Linux 9
pveam download rockylinux-9 standard 9

# AlmaLinux 9
pveam download almalinux-9 standard 9

# Configure
pct exec CTID -- dnf update -y
pct exec CTID -- dnf install -y epel-release
```

---

## Custom Templates

### Create Custom Template

```bash
# 1. Create container
pct create 999 local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:4 \
  --hostname custom-template \
  --memory 1024 --cores 1

# 2. Configure
pct exec 999 -- bash -c "apt update && apt upgrade -y"
pct exec 999 -- apt install -y software-properties-common
pct exec 999 -- apt remove -y --purge *-recommends

# 3. Clean
pct exec 999 -- apt clean
pct exec 999 -- rm -rf /var/cache/apt/archives/*
pct exec 999 -- rm -rf /tmp/*

# 4. Create template
pct template 999

# 5. Export
tar -czf /var/lib/vz/template/cache/custom-debian12.tar.gz \
  -C /var/lib/lxc/999/rootfs .
```

### Template from Scratch

```bash
# Create minimal Debian
debootstrap bookworm /mnt/root http://deb.debian.org/debian

# Configure
echo "debian" > /mnt/root/etc/hostname
echo "127.0.0.1 debian" >> /mnt/root/etc/hosts

# Create tarball
tar -czf /var/lib/vz/template/cache/minimal-debian.tar.gz \
  -C /mnt/root .
```

### Template Variables

```bash
# Common variables
ARCH=$(uname -m)
DISTRO=debian
VERSION=12
CODENAME=bookworm
```

---

## Template Variables

### Environment Variables

```bash
# In template script
$ARCH          # Architecture
$DISTRO        # Distribution
$VERSION      # Version
$CODENAME     # Code name
$BUILD        # Build number
```

### Configuration Variables

```bash
# CT configuration
$CTID         # Container ID
$HOSTNAME     # Container hostname
$PASSWORD    # Root password
$STORAGE      # Storage location
$DISK_SIZE    # Disk size
$RAM         # Memory
$CORES       # CPU cores
```

### Network Variables

```bash
# Network
$IP          # IP address
$GW          # Gateway
$NETMASK     # Netmask
$DNS         # DNS servers
$DOMAIN      # Domain
```

---

## Autoinstall

### Ubuntu Autoinstall

```yaml
# user-data
autoinstall:
  version: 1
  locale: en_US.UTF-8
  identity:
    hostname: ubuntu
    password: "$ hashed password"
    username: ubuntu
  ssh:
    install-server: yes
    allow-pw: yes
  storage:
    layout:
      name: lvm
  packages:
    - openssh-server
    - vim
```

### Fedora Autoinstall

```bash
# Download Fedora ISO with autoinstall
wget https://download.fedoraproject.org/pub/fedora/linux/releases/39/Server/x86_64/iso/Fedora-39-x86_64-iso.torrent

# Use kickstart
# ks=file:/ks.cfg
```

### RHEL/Alma/Rocky Autoinstall

```bash
# AlmaLinux kickstart
# /ks.cfg
lang en_US.UTF-8
keyboard us
timezone UTC --isUtc
rootpw --iscrypted $encryptedpassword
network --bootproto=dhcp --device=eth0
bootloader --location=mbr
clearpart --all --initlabel
part /boot --fstype=xfs --ondisk=sda
part pv.01 --grow --ondisk=sda
volgroup vg_root pv.01
logvol / --fstype=xfs --size=8192 --name=lv_root --vgname=vg_root
logvol swap --size=2048 --name=lv_swap --vgname=vg_root
logvol /home --fstype=xfs --grow --name=lv_home --vgname=vg_root
%packages
@^minimal-environment
openssh-server
%end
```

---

## Windows Templates

### Windows ISOs

```bash
# Download Windows ISO
#.microsoft.com/software-download/windows11

# Add to Proxmox
pvesm add dir local --path /var/lib/vz/template/iso

# Upload ISO
# Web UI: Storage → ISO Images → Upload
```

### VirtIO Drivers

```bash
# Download VirtIO drivers
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win-0.1.XX.iso

# Add as CDROM
qm set VMID --ide2 local:iso/virtio-win.iso,media=cdrom

# Install drivers in Windows:
# Device Manager → Update drivers → virtio-win.iso
```

### Windows Setup

```bash
# Create VM for Windows 11
qm create 100 \
  --name "windows-11" \
  --memory 4096 \
  --cores 4 \
  --cpu host \
  --ostype win10 \
  --bios ovmf \
  --scsi0 local:vm-100-disk-0,size=64G,discard=1,ssd=1 \
  --net0 virtio,bridge=vmbr0 \
  --cdrom local:iso/windows-11.iso

# Install VirtIO after Windows
# Device Manager → Update drivers
```

### Windows Evaluation

```bash
# Evaluation versions:
# Windows Server 2025: 180 days
# Windows 11 Enterprise: 90 days

# Convert to VHDX
qm disk import 100 /path/to/windows.vhdx local
```

---

## Quick Reference

### Template Commands

```bash
# List templates
pveam list standard

# Download template
pveam download debian-12 standard 12

# Create container from template
pct create 200 local:vztmpl/debian-12.tar.gz --rootfs local:8

# Create template from container
pct template 200

# Clone VM as template
qm template 100
```

### Template Storage

```bash
# Store templates
/var/lib/vz/template/cache/     # LXC templates
/var/lib/vz/template/iso/       # ISO images
/var/lib/vz/template/cdlocal/   # Custom CD-ROM
```

### Recommended Settings

| Template | RAM | Cores | Disk |
|----------|-----|-------|------|
| Debian | 512MB | 1 | 2GB |
| Ubuntu | 512MB | 1 | 2GB |
| Alpine | 128MB | 1 | 1GB |
| CentOS | 512MB | 1 | 4GB |
| Windows | 2GB | 2 | 30GB |

## Keywords

#templates #lxc #vm #cloud-init #autoinstall #custom #iso

## Links

- [[Proxmox-VE]] - Main documentation
- [[Installation]] - Installation guide
- [[Containers]] - Container management
- [[VM]] - VM management
- [[HomeLab]] - Home lab setup
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
