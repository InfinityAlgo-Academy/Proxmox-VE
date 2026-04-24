# How to Test Backup and Restore - Complete Guide

## Question: How do I ensure my backups actually work?

### Answer: Regular testing is critical

---

## Why Test Backups

### Statistics
- 40% of backup restores fail
- Only 35% of companies test quarterly
- 70% of failures are due to untested backups

---

## How to Test Monthly

### Step 1: Create Test Environment
```bash
# Create a test VM to restore into
qm create 999 --name test-restore --memory 2048 --cores 2
```

### Step 2: Find Latest Backup
```bash
# List backups
ls -lt /var/lib/vz/dump/ | head -5

# Note the filename
```

### Step 3: Restore to Test VM
```bash
# Restore backup
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2024_*.tar.gz 999

# Or for container
pct restore 299 /var/lib/vz/dump/vzdump-lxc-200-2024.tar.gz
```

### Step 4: Verify Test VM Starts
```bash
# Start the test VM
qm start 999

# Wait for boot
sleep 30

# Check status
qm status 999
```

### Step 5: Verify Data Integrity
```bash
# Check specific files exist
qm exec 999 -- ls -la /var/log/

# Or test service
qm exec 999 -- systemctl status nginx
```

### Step 6: Clean Up
```bash
# Stop and destroy test VM
qm stop 999 --forceStop 1
qm destroy 999 --destroy-unlinked
```

---

## How to Use Automation

### Test Script
```bash
#!/bin/bash
# test-restore.sh

TEST_VM=998
BACKUP=$(ls -t /var/lib/vz/dump/vzdump-qemu-100-*.tar.gz | head -1)

echo "Testing: $BACKUP"

qmrestore "$BACKUP" $TEST_VM
qm start $TEST_VM
sleep 30

if qm status $TEST_VM | grep -q running; then
    echo "TEST PASSED"
    qm stop $TEST_VM
    qm destroy $TEST_VM
    exit 0
else
    echo "TEST FAILED"
    exit 1
fi
```

---

## Keywords

#test-backup #restore #how-to #backup-testing #verification

## Related Articles

- [[Backup]]
- [[How-To-Restore]]
- [[How-To-Test-Backup]]
- [[Proxmox-VE]]