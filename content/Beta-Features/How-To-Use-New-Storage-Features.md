# How to Use New Storage Features - Complete Guide

## Question: How do I test new storage features?

### Answer: Available storage betas

---

## How to Enable New Storage Options

### Step 1: Enable Experimental Features
```bash
# Enable feature flags
pvesh put /cluster/options --experimental-features 1
```

---

## How to Use LVM Thin v2

### Step 1: Create Thin Pool v2
```bash
lvcreate -L 1T -T --type thin-pool vg0/thin-pool-v2
```

---

## How to Use ZFS New Features

### Step 1: Enable Deduplication v2
```bash
zfs set dedup=lz4 pool1
zfs set checksum=sha256 pool1
```

---

## Keywords

#storage #lvm #zfs #how-to

## Related Articles

- [[Beta-Features]]
- [[Storage]]
- [[LVM-Thin]]
- [[Proxmox-VE]]