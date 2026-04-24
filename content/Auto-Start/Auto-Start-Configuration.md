# Auto-Start Configuration Complete Guide

## Overview

Auto-start automatically starts VMs and containers after Proxmox VE boots. Essential for HA setups and ensuring services start automatically after host reboot.

## Configure Auto-Start

### Via Web UI

1. Navigate to **Datacenter → VM/Container**
2. Select VM → **Options** tab
3. Find **Start at boot** setting
4. Enable/disable

### Via CLI

```bash
# Enable auto-start for VM
qm set 100 --onboot 1

# Disable auto-start for VM
qm set 100 --onboot 0

# Enable for container
pct set 200 --onboot 1
```

## Boot Order

### For VMs

```bash
# Set boot order
qm set 100 --boot order=scsi0

# Multiple boot devices
qm set 100 --boot order=scsi0;ide2

# DVD boot first
qm set 100 --boot order=ide2;scsi0
```

### Boot Options

| Parameter | Description |
|-----------|-------------|
| order=scsi0 | Boot from SCSI disk |
| order=ide2 | Boot from DVD |
| order=net0 | Network (PXE) |
| order=usb | USB device |

## Boot Delay

### Configure Delay

```bash
# Add boot delay (seconds)
qm set 100 --boot delay=5

# Useful for:
# - Storage not immediately available
# - Hardware initialization
```

## Startup and Shutdown Behavior

### Startup Delay

```bash
# Delay between VM startups
# File: /etc/pve/datacenter.conf
# Add startup delay
pve config set --startup-delay 5

# View current config
cat /etc/pve/datacenter.conf
```

### Shutdown Timeout

```bash
# Graceful shutdown timeout
qm set 100 --shutdown-timeout 120

# Default: 180 seconds
# For slow shutdown: increase
```

## Keywords

#auto-start #boot-order #onboot #startup #boot-delay

## Related Articles

- [[Auto-Start]]
- [[VM-Configuration]]