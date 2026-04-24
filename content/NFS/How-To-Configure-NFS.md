---
title: How to Configure NFS Storage - Complete Beginner's Guide
---

# How to Configure NFS Storage - Complete Beginner's Guide

## Question: How do I set up NFS storage?

### Answer: Follow these steps

---

## How to Add NFS to Proxmox

### Step 1: Create NFS Share (Server)
```bash
apt install nfs-kernel-server
mkdir -p /mnt/nfsshare
echo "/mnt/nfsshare 192.168.1.0/24(rw,sync)" >> /etc/exports
exportfs -a
```

### Step 2: Add NFS in Proxmox
1. **Datacenter** → **Storage** → **Add** → **NFS**
2. Server: 192.168.1.200
3. Export: /mnt/nfsshare
4. ID: nfs-share

### Step 3: CLI Method
```bash
pvesm add nfs nfs-share \
  --server 192.168.1.200 \
  --export /mnt/nfsshare \
  --content vztmpl,iso,backup
```

---

## Keywords

#nfs #storage #how-to #beginner

## Related Articles

- [[NFS]]
- [[Storage]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
