# Cluster Boot Management

## Overview

Boot management in Proxmox cluster ensures proper VM startup order across all nodes. Critical for HA environments and maintenance.

## Cluster Boot Configuration

### Global Boot Options

```bash
# View cluster options
pvesh get /cluster/options

# Set startup delay
pvesh put /cluster/options --startup-delay 30

# Set shutdown timeout
pvesh put /cluster/options --shutdown-timeout 300
```

### Boot Order in Cluster

```bash
# Check boot order across cluster
pvecm status

# List VMs by node
qm list
```

## Node Boot Order

### Boot Sequence

```bash
# Node boots first
# Then VMs based on:
# 1. onboot setting
# 2. startup-delay
# 3. HA priority
```

### Configure Per Node

```bash
# Set onboot for VMs on specific node
for vm in 100 101 102; do
  qm set $vm --onboot 1 --startup-delay 10
done
```

## HA Boot Management

### Enable HA Boot

```bash
# Enable HA
ha-manager enable vm:100

# Disable HA
ha-manager disable vm:100
```

### HA Priority

```bash
# Set priority
ha-manager crm add production --vm 100 --priority 100
```

## Cluster Shutdown

### Graceful Shutdown

```bash
# Shutdown all VMs first
pvecm cluster shutdown --force

# Shutdown with timeout
pvecm cluster shutdown --timeout 600
```

### Individual Node Boot

```bash
# Boot single node
qm start 100 --pve2

# Start container
pct start 200 --pve2
```

## Related Articles

- [[Auto-Start]]
- [[Cluster]]
- [[High-Availability]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
