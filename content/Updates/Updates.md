# Proxmox VE Updates Guide

## Table of Contents

1. [Update Process](#update-process)
2. [Release Notes](#release-notes)
3. [Rollback](#rollback)
4. [Testing Updates](#testing-updates)

---

## Update Process

```bash
# Update Proxmox
apt update && apt upgrade -y

# Update container templates
pveam update

# Update Ceph
pveceph update

# Reboot
reboot
```

### Scheduled Updates

```bash
# Install unattended-upgrades
apt install -y unattended-upgrades

# Configure
dpkg-reconfigure -plow unattended-upgrades
```

---

## Release Notes

### Version 8.3

### Version 8.2

### Version 8.1

---

## Rollback

```bash
# List available kernels
dpkg --list | grep linux-image

# Boot previous kernel
# Select from GRUB menu

# Hold current kernel
apt-mark hold linux-image-6.8.12-generic
```

---

## Testing Updates

```bash
# Use test repository
echo "deb http://download.proxmox.com/debian/pve bookworm pvetest" > /etc/apt/sources.list.d/pvetest.list
apt update && apt upgrade -y
```

---

## Keywords

#updates #upgrade #release #rollback

## Links

- [[Proxmox-VE]] - Main documentation
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
