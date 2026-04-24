# How to Troubleshoot Common Issues - Complete Beginner's Guide

## Question: How do I fix common Proxmox problems?

### Answer: Solutions for common issues

---

## Problem 1: VM Won't Start

### Solution
```bash
# Check error
qm start 100

# Check storage
pvesm status

# Check memory
qm set 100 --memory 2048
```

---

## Problem 2: No Network

### Solution
```bash
# Restart network
systemctl networking restart

# Check bridge
ip link show vmbr0
```

---

## Problem 3: Storage Not Available

### Solution
```bash
# Enable storage
pvesm enable local

# Check status
pvesm status
```

---

## Problem 4: Cannot Access Web UI

### Solution
```bash
# Check service
systemctl status pveproxy

# Restart
systemctl restart pveproxy

# Check firewall
```

---

## Problem 5: Low Disk Space

### Solution
```bash
# Check usage
df -h /var/lib/vz

# Clean old files
rm -rf /var/lib/vz/dump/*
```

---

## Keywords

#troubleshooting #how-to #beginner #error

## Related Articles

- [[Troubleshooting]]
- [[Proxmox-VE]]