# Startup Delay and Timing Configuration

## Overview

Startup delay controls the timing between VM boots, storage availability, and network configuration. Proper timing prevents boot failures during host startup.

## Configure Startup Delay

### Between VMs

```bash
# File: /etc/pve/datacenter.conf
# Set delay between VM startups (seconds)
pvesh put /cluster/options --startup-delay 30

# View current setting
pvesh get /cluster/options | grep startup
```

### Per VM Delay

```bash
# Additional delay for specific VM
qm set 100 --startup-delay 30
```

## Storage Wait Time

### Wait for Storage

```bash
# Wait for storage to be available
pvesh put /cluster/options --storage-start-delay 10
```

## Network Delay

### Wait for Network

```bash
# Network startup timeout
# File: /etc/systemd/network/50-default.network
[Network]
DHCP=ipv4

[DHCP]
RouteMetric=100
```

## Boot Timeout

### VM Shutdown Timeout

```bash
# Time to wait for graceful shutdown
qm set 100 --shutdown-timeout 180

# Default: 180 seconds
```

## Timing Examples

### Fast Boot

```bash
# Minimal delays
qm set 100 --startup-delay 0
pvesh put /cluster/options --startup-delay 5

# Total boot time: ~10 seconds per VM
```

### Reliable Boot

```bash
# Extended delays
qm set 100 --startup-delay 30
pvesh put /cluster/options --startup-delay 30

# For:
# - Slow storage
# - Network boot
# - Hardware issues
```

## Troubleshooting Timing

### VM Starts Before Storage

```bash
# Increase storage wait
pvesh put /cluster/options --storage-start-delay 20
```

### Network Not Ready

```bash
# Check network
networkctl

# Enable wait for network
systemctl enable systemd-networkd-wait-online
```

## Related Articles

- [[Auto-Start]]
- [[Storage-Configuration]]
- [[Network-Config]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
