---
title: VLAN Configuration
tags:
  - vlan
  - network
  - networking
  - tagging
---

# VLAN Configuration

## Overview

VLANs (Virtual Local Area Networks) segment your network for isolation and security.

## Create VLAN

### Method 1: Via /etc/network/interfaces

```bash
auto vmbr0.100
iface vmbr0.100 inet static
    address 192.168.100.1
    netmask 255.255.255.0
    vlan-raw-device vmbr0
```

### Method 2: Via ip command

```bash
# Create VLAN interface
ip link add link vmbr0 name vmbr0.100 type vlan id 100

# Set IP
ip addr add 192.168.100.1/24 dev vmbr0.100

# Enable
ip link set vmbr0.100 up
```

## VLAN for VMs

```bash
# Create VM with VLAN
qm create 100 --net0 virtio,bridge=vmbr0,tag=100
```

## VLAN Ranges

```bash
# Allow VLAN range
iface vmbr0 inet static
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

## Commands

| Command | Description |
|---------|-------------|
| `ip link show type vlan` | Show VLANs |
| `bridge vlan show` | Show VLAN ports |

## Troubleshooting

- VLAN not working: Check switch port is trunk
- No connectivity: Verify VLAN ID

## See Also

- [[index|Back to Proxmox VE]]
- [[Network-Config]]
- [[Network-Segmentation]]