---
title: Network Bonding
tags:
  - network
  - bonding
  - lacp
  - failover
---

# Network Bonding

## Overview

Network bonding combines multiple network interfaces for redundancy and increased bandwidth.

## Bond Modes

| Mode | Name | Features |
|------|------|-----------|
| 0 | balance-rr | Round-robin |
| 1 | active-backup | Active/passive failover |
| 2 | balance-xor | Hash-based |
| 3 | broadcast | All ports |
| 4 | 802.3ad (LACP) | Link aggregation |
| 5 | balance-tlb | Transmit load balance |
| 6 | balance-alb | Adaptive load balance |

## Create Bond (LACP)

```bash
# /etc/network/interfaces
auto bond0
iface bond0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    slaves eth0 eth1
    bond-mode 4
    bond-miimon 100
    bond-lacp-rate 1
    bond-active-slaeve eth0 eth1
```

## Verify Bond

```bash
# Check bond status
cat /proc/net/bonding/bond0

# Show interfaces
ip link show
```

## Bond with Bridge

```bash
auto bond0
iface bond0 inet static
    bond-mode 4
    bond-miimon 100

auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    bridge-ports bond0
    bridge-stp off
```

## Troubleshooting

- LACP not working: Check switch port
- No failover: Verify mode 1

## See Also

- [[index|Back to Proxmox VE]]
- [[Networking]]
- [[Network-Config]]