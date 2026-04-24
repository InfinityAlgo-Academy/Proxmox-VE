# Startup Errors and Fixes Complete Guide

## Overview

Complete guide to fixing all VM startup errors. Learn to diagnose and resolve common startup issues quickly.

## Error Reference

### Error 1: VM Already Running

**Message:**
```
vm is already running
```

**Cause:**
- VM started on another node
- Stale lock file

**Fix:**
```bash
# Check where VM is running
qm list

# If running elsewhere, stop there
qm stop 100

# Or remove stale lock
rm /var/lib/vz/images/100/lock
```

### Error 2: Storage Not Available

**Message:**
```
storage 'local' is not available
```

**Cause:**
- Storage not mounted
- Storage offline

**Fix:**
```bash
# Check storage
pvesm status

# Enable storage
pvesm enable local

# Check storage content
pvesm list local
```

### Error 3: Not Enough Disk Space

**Message:**
```
no space left on device
```

**Cause:**
- Storage full

**Fix:**
```bash
# Check space
pvesm status

# Clean up old backups
rm -rf /var/lib/vz/dump/*

# Expand storage
pvesm resize local +10G
```

### Error 4: CPU Not Available

**Message:**
```
cpu hotplug failed
```

**Cause:**
- No available CPU cores

**Fix:**
```bash
# Check CPU
pvesh get /nodes/localhost/status

# Reduce VM CPU
qm set 100 --cores 2
```

### Error 5: Memory Not Available

**Message:**
```
not enough memory
```

**Cause:**
- RAM exhausted

**Fix:**
```bash
# Check memory
free -h

# Reduce VM memory
qm set 100 --memory 2048

# Enable balloon
qm set 100 --balloon 2048
```

### Error 6: Network Not Available

**Message:**
```
network 'vmbr0' does not exist
```

**Cause:**
- Bridge not configured

**Fix:**
```bash
# Check bridges
brctl show

# Create bridge
ip link add vmbr0 type bridge

# Or use existing
qm set 100 --net0 virtio,bridge=vmbr0
```

### Error 7: Lock Timeout

**Message:**
```
could not acquire lock
```

**Cause:**
- Another operation in progress

**Fix:**
```bash
# Check locks
qm list

# Wait for lock
sleep 30

# Force unlock (dangerous)
qm unlock 100
```

### Error 8: Invalid Configuration

**Message:**
```
invalid configuration
```

**Cause:**
- Wrong settings

**Fix:**
```bash
# Check config
qm config 100

# Reset to defaults
qm set 100 --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0
```

### Error 9: ISO Not Found

**Message:**
```
iso 'local:iso/xxx.iso' does not exist
```

**Cause:**
- ISO not uploaded

**Fix:**
```bash
# List ISOs
pvesm list local

# Upload new ISO
pvesm upload local iso/ubuntu.iso

# Or remove from config
qm set 100 --ide2 none
```

### Error 10: Permission Denied

**Message:**
```
permission check failed
```

**Cause:**
- User lacks permissions

**Fix:**
```bash
# Check ACL
pveum acl list

# Grant permission
pveum acl modify /vms/100 --user admin@example.com --role PVEPowerAdmin
```

## Emergency Recovery

### Force Start

```bash
# Force start
qm start 100 --force 1

# Skip lock
qm start 100 --skiplocking 1
```

### Reset Configuration

```bash
# Reset VM
qm destroy 100
qm create 100
```

## Prevention

### Health Checks

```bash
# Pre-boot check
qm start 100 --verify
```

## Related Articles

- [[Auto-Start]]
- [[Troubleshooting]]
- [[Boot-Problems]]