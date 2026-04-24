# How to Handle Backup Failures - Complete Guide

## Question: My backup failed - how do I fix it?

### Answer: Identify and fix the issue

---

## Common Error Messages and Fixes

### Error 1: "Storage not available"

**Cause:** Storage disconnected

**Fix:**
```bash
# Check storage status
pvesm status

# Fix storage
pvesm enable local

# Check storage connectivity
ping storage-server-ip
```

---

### Error 2: "No space left on device"

**Cause:** Storage full

**Fix:**
```bash
# Check storage usage
pvesm status

# Clean old backups
rm /var/lib/vz/dump/vzdump-*.tar.gz

# Configure retention
# Web UI -> Backup -> Job -> Retention -> Set lower values
```

---

### Error 3: "VM is locked"

**Cause:** VM in use by another process

**Fix:**
```bash
# Check VM status
qm list

# Wait for process or unlock
qm unlock 100
```

---

### Error 4: "Permission denied"

**Cause:** User lacks backup permission

**Fix:**
```bash
# Check permissions
pveum acl list

# Grant backup permission
pveum acl modify /vms/100 --user admin@pam --role PVEPowerAdmin
```

---

### Error 5: "Backup already running"

**Cause:** Previous backup still running

**Fix:**
```bash
# Wait for completion
# Or kill backup process
pkill vzdump
```

---

## How to Debug Backup Issues

### Enable Verbose Logging

```bash
# Run backup with verbose output
vzdump 100 --mode snapshot --verbose

# Check system logs
tail -f /var/log/pveproxy/access.log
```

---

## How to Prevent Future Failures

### Best Practices

1. **Monitor** storage space weekly
2. **Set retention** properly
3. **Check logs** regularly
4. **Test restore** quarterly

---

## Keywords

#troubleshooting #error #how-to #debug

## Related Articles

- [[Backup]]
- [[How-To-Configure-Backup]]
- [[Troubleshooting]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
