---
title: OnBoot Option Deep Dive
---

# OnBoot Option Deep Dive

## Overview

The onboot option controls whether a VM or container starts automatically when the Proxmox VE host boots. This is critical for production services.

## Enable OnBoot

### For VMs

```bash
# Enable onboot
qm set 100 --onboot 1

# Disable
qm set 100 --onboot 0

# Check status
qm config 100 | grep onboot
```

### For Containers

```bash
# Enable onboot
pct set 200 --onboot 1

# Disable
pct set 200 --onboot 0
```

## OnBoot in Cluster

### HA Considerations

```bash
# HA managed VMs
# onboot is handled by HA
ha-manager status
```

### Priority with OnBoot

```bash
# Set startup priority
qm set 100 --onboot 1 --startup-delay 10

# Lower number = higher priority
# Default: 0
```

## Troubleshooting OnBoot

### VM Not Starting

```bash
# Check onboot status
qm config 100 | grep onboot

# Verify VM exists
qm list

# Check host status
pvesh get /nodes/localhost/status
```

### Start Order Wrong

```bash
# Set startup order
qm set 100 --onboot 1 --startup-delay 0
qm set 101 --onboot 1 --startup-delay 10
qm set 102 --onboot 1 --startup-delay 20
```

## Related Articles

- [[Auto-Start]]
- [[Auto-Start-Configuration]]
---

[[index|Back to Proxmox VE]]
