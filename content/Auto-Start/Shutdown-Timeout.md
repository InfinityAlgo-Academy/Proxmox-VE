# Shutdown Timeout Configuration

## Overview

Shutdown timeout determines how long Proxmox waits for a VM to gracefully shut down before forcing a stop. Critical for data integrity and proper VM lifecycle.

## Configure Shutdown Timeout

### Default Timeout

```bash
# Default: 180 seconds
qm config 100 | grep shutdown-timeout
```

### Set Custom Timeout

```bash
# Set 5 minute timeout
qm set 100 --shutdown-timeout 300

# Quick shutdown (1 minute)
qm set 100 --shutdown-timeout 60

# Extended shutdown (10 minutes)
qm set 100 --shutdown-timeout 600
```

## Timeout by OS

### Windows

```bash
# Windows needs more time
qm set 100 --shutdown-timeout 300
```

### Linux

```bash
# Linux typically fast
qm set 100 --shutdown-timeout 120
```

### Application VMs

```bash
# Database/proxy VMs
qm set 100 --shutdown-timeout 600
```

## Shutdown Process

### Graceful Shutdown

```bash
# Initiates ACPI
qm stop 100

# Sends shutdown signal
# Waits for timeout
```

### Force Stop

```bash
# Immediate stop
qm stop 100 --forceStop 1

# Or
qm stop 100 --skiplocking 1
```

## Troubleshooting Shutdown

### VM Won't Shutdown

```bash
# Check shutdown status
qm monitor 100

# Force stop if needed
qm stop 100 --forceStop 1

# Then investigate
qm cleanup 100
```

### Timeout Too Short

```bash
# Increase timeout
qm set 100 --shutdown-timeout 600
```

## Global Shutdown Options

### Cluster Shutdown

```bash
# Cluster-wide
pvecm cluster shutdown

# Timeout
pvecm cluster shutdown --timeout 300
```

## Related Articles

- [[Auto-Start]]
- [[VM-Management]]
---

[[index|Back to Proxmox VE]]
