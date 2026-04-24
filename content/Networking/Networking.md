---
title: Network Configuration
tags:
  - network
  - networking
  - bridge
  - configuration
---

# Network Configuration

## Overview

Proxmox VE supports flexible networking through Linux bridges, bonds, VLANs, and more.

## Types of Networks

### 1. Bridge Network

The most common network type - connects VMs to physical network.

```bash
# Create bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
```

### 2. Bond Network

Link aggregation for redundancy and bandwidth.

```bash
# Create bond
auto bond0
iface bond0 inet static
    slaves eth0 eth1
    bond-mode 802.3ad
    bond-miimon 100
    bond-lacp-rate 1
```

### 3. VLAN Network

802.1q tagging for network segmentation.

```bash
# Create VLAN
auto vmbr0.100
iface vmbr0.100 inet static
    address 192.168.100.1
    netmask 255.255.255.0
    vlan-raw-device vmbr0
```

## Network Commands

| Command | Description |
|---------|-------------|
| `ip addr` | Show addresses |
| `ip link` | Show interfaces |
| `ip route` | Show routes |
| `brctl show` | Show bridges |
| `bridge link` | Show bridge ports |

## VM Network Configuration

```bash
# Create VM with network
qm create 100 --name "web-server" \
  --net0 virtio,bridge=vmbr0,firewall=1

# Update network
qm set 100 --net0 virtio,bridge=vmbr0
```

## Container Network

```bash
# Create container with network
pct create 200 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Set static IP
pct set 200 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.50/24,gw=192.168.1.1
```

## Troubleshooting

- [[Troubleshooting-Network]]
- [[Network-Monitoring]]
- [[Network-Bonding]]
- [[VLAN-Config]]

## See Also

- [[index|Back to Proxmox VE]]