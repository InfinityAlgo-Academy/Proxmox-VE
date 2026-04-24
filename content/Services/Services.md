# Proxmox VE Services Guide

## Table of Contents

1. [System Services](#system-services)
2. [Network Services](#network-services)
3. [Storage Services](#storage-services)
4. [VM Services](#vm-services)
5. [Container Services](#container-services)

---

## System Services

### Proxmox Services

```bash
# Proxmox VE GUI
systemctl status pveproxy

# Proxmox Cluster
systemctl status corosync
systemctl status pvecm

# Proxmox HA
systemctl status pveha-lrm
systemctl status pveha-cluster
```

### Control Services

```bash
# Start service
systemctl start <service>

# Stop service
systemctl stop <service>

# Restart service
systemctl restart <service>

# Enable at boot
systemctl enable <service>

# Disable at boot
systemctl disable <service>
```

---

## Network Services

### DNS Services

```bash
# dnsmasq
systemctl status dnsmasq
systemctl enable dnsmasq
```

### DHCP Services

```bash
# isc-dhcp-server
systemctl status isc-dhcp-server
systemctl enable isc-dhcp-server
```

---

## Storage Services

### NFS Services

```bash
# nfs-kernel-server
systemctl status nfs-kernel-server
systemctl enable nfs-kernel-server
```

### iSCSI Services

```bash
# iscsid
systemctl status iscsid
systemctl enable iscsid
```

---

## VM Services

### QEMU Guest Agent

```bash
# Install
apt install -y qemu-guest-agent

# Service status
systemctl status qemu-guest-agent
```

---

## Container Services

### Docker in Container

```bash
# Docker
systemctl status docker
systemctl enable docker
```

### LXD

```bash
# LXD
systemctl status lxd
systemctl enable lxd
```

---

## Keywords

#services #systemd #daemon

## Links

- [[Proxmox-VE]] - Main documentation
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
