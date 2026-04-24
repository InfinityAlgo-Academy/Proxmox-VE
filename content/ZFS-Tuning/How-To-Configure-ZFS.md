# How to Configure ZFS Storage - Complete Beginner's Guide

## Question: How do I set up ZFS?

### Answer: Follow these steps

---

## How to Create ZFS Pool

### Step 1: Create Pool
1. **Datacenter** → **Storage** → **Add** → **ZFS**
2. Name: pool-name
3. Pool: /dev/disk/by-id/xxx

### Step 2: CLI Method
```bash
# Create pool
zpool create -f pool1 mirror /dev/sda /dev/sdb

# Check
zpool status
```

---

## How to Set Compression

```bash
zfs set compression=lz4 pool1
```

---

## Keywords

#zfs #storage #how-to #beginner

## Related Articles

- [[ZFS-Tuning]]
- [[Storage]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
