---
title: Network Segmentation
tags:
  - network
  - segmentation
  - vlan
  - isolation
  - security
---

# Network Segmentation

## Overview

Network segmentation isolates different types of traffic for security and performance.

## Segmentation Strategies

### 1. VLAN Segmentation

```bash
# Management VLAN
auto vmbr0.10
iface vmbr0.10 inet static
    address 192.168.10.1/24

# VM VLAN
auto vmbr0.20
iface vmbr0.20 inet static
    address 192.168.20.1/24

# Guest VLAN
auto vmbr0.30
iface vmbr0.30 inet static
    address 192.168.30.1/24
```

### 2. Separate Physical Networks

```bash
# Management network
auto vmbr0
iface vmbr0 inet static
    address 192.168.10.100/24

# Production network  
auto vmbr1
iface vmbr1 inet static
    address 192.168.20.100/24
```

### 3. Network Isolation

```bash
# Isolated network (no internet)
auto vmbr2
iface vmbr2 inet static
    address 10.0.0.1/24
    bridge-ports none
```

## Use Cases

| Network | Purpose | VLAN |
|---------|---------|------|
| Management | Proxmox management | 10 |
| Production | Production VMs | 20 |
| Development | Dev VMs | 30 |
| Guest | Guest network | 40 |
| Backup | Backup traffic | 50 |

## Firewall Rules

```bash
# Allow only specific traffic between segments
iptables -A FORWARD -i vmbr0.10 -o vmbr0.20 -j ACCEPT
iptables -A FORWARD -i vmbr0.20 -o vmbr0.10 -j REJECT
```

## Best Practices

1. Use separate VLANs for different purposes
2. Restrict inter-VLAN traffic with firewall
3. Monitor cross-VLAN traffic
4. Use strong authentication

## See Also

- [[index|Back to Proxmox VE]]
- [[VLAN-Config]]
- [[Firewall-Rules]]
- [[Security]]