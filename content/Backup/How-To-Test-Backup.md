# How to Test Backup Works - Complete Guide

## Question: How do I verify my backups are working?

### Answer: Test restoration is essential

---

## Method 1: Using Web UI

### Step-by-Step:

**1. Navigate to Storage**
- Go to **Datacenter** → **Storage** → **local**
- Click **Content**

**2. Verify Backup File**
- Find your backup file
- Check file size (should be > 0)

**3. Check Backup Log**
- Go to **Datacenter** → **Backup**
- Look at job status (shows success/failed)

---

## Method 2: Using CLI

### Test Backup Integrity

```bash
# List backups
vzdump --list

# Verify backup file
tar -tzf /var/lib/vz/dump/vzdump-qemu-100-*.tar.gz | wc -l

# Check file size
ls -lh /var/lib/vz/dump/
```

---

## How to Test Restore (Test VM)

### Test Restore Steps:

**1. Create Test VM**
```bash
# Create test VM for restore test
qm create 999 --name test-restore --memory 1024
```

**2. Restore to Test VM**
```bash
# Restore backup to test VM
qmrestore /var/lib/vz/dump/vzdump-*.tar.gz 999
```

**3. Verify Test VM**
```bash
# Check test VM
qm config 999

# Start if needed
qm start 999
```

**4. Clean Up**
```bash
# Destroy test VM
qm destroy 999 --destroy-unlinked
```

---

## How to Schedule Test Restore

### Monthly Test Schedule

```bash
# Add to calendar - monthly first Sunday
# Manually schedule monthly test
```

### Automation Script

```bash
#!/bin/bash
# test-restore.sh - Test restore monthly

TEST_VM=998
BACKUP_FILE=$(ls -t /var/lib/vz/dump/vzdump-qemu-100*.tar.gz | head -1)

echo "Testing backup: $BACKUP_FILE"

# Restore
qmrestore "$BACKUP_FILE" $TEST_VM

# Check VM
qm start $TEST_VM

# Wait and check
sleep 30

# Get VM status
if qm status $TEST_VM | grep -q running; then
    echo "TEST PASSED: VM running"
else
    echo "TEST FAILED: VM not running"
    exit 1
fi

# Cleanup
qm stop $TEST_VM --forceStop 1
qm destroy $TEST_VM

echo "Test complete"
```

---

## Pro Tips

1. **Test monthly** - At minimum quarterly
2. **Document** test results
3. **Keep test VM** template ready

---

## Keywords

#test #restore #verification #how-to #beginner

## Related Articles

- [[Backup]]
- [[How-To-Restore]]
- [[How-To-Configure-Backup]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
