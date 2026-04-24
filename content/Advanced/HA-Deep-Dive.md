---
title: High Availability Deep Dive
---

# High Availability Deep Dive

## Overview

Proxmox VE High Availability (HA) provides automatic VM/container failover when a node fails. Essential for production workloads requiring uptime.

## HA Architecture

### Components

- **HA Manager**: Coordinates failover
- **Fencing**: Power cycles failed nodes
- **Resource Groups**: Priority VM placement

### Quorum

```bash
# Check quorum
pvecm status

# View expected
pvecm e
```

## Configure HA

### Enable HA for VM

```bash
# Enable HA
ha-manager enable vm:100

# Set priority
ha-manager crm add production --vm 100 --priority 100
```

### Configure Fencing

```bash
# Install fence agents
apt install fence-agents-all

# Configure via Web UI
# Datacenter -> HA -> Fencing
```

## HA Groups

### Create Group

```bash
# Create group
ha-manager group add production pve1 pve2 pve3

# Add node with priority
ha-manager group add production pve1 pve2 pve3

# Remove node
ha-manager group remove production pve3
```

### Node Priority

```bash
# Set priority
ha-manager crm modify vm:100 --priority 100
```

## HA Best Practices

1. Minimum 3 nodes
2. Separate power sources
3. Configure fencing
4. Test failover

## Keywords

#ha #high-availability #failover #cluster #quorum #fencing

## Related Articles

- [[Advanced]]
- [[Cluster]]
- [[High-Availability]]
---

[[index|Back to Proxmox VE]]
