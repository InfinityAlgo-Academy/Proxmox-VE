# How to Prepare for Disaster Recovery - Complete Guide

## Question: How do I prepare for disaster recovery?

### Answer: Document and prepare ahead

---

## How to Document Configuration

### Step 1: Export All Configs
```bash
# VM configs
qm config 100 > /backup/vm-100-config.txt
qm config 101 > /backup/vm-101-config.txt

# Container configs
pct config 200 > /backup/ct-200-config.txt

# Network config
cp /etc/network/interfaces /backup/network-config.txt
```

---

## How to Create Recovery Plan

### Step 1: Write Document
Include:
- Server hardware details
- Network topology
- IP addresses
- Backup locations
- Recovery procedures
- Contact information

### Step 2: Test Recovery
```bash
# Monthly test restore
./test-restore.sh
```

---

## How to Store Recovery Info

### Step 1: Offsite Documentation
```bash
# Export to USB
cp /backup/* /mnt/usb/recovery/

# Sync to cloud
rclone sync /backup cloud:recovery-docs
```

---

## Keywords

#disaster-recovery #documentation #how-to #backup

## Related Articles

- [[Best-Practices]]
- [[Disaster-Recovery]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
