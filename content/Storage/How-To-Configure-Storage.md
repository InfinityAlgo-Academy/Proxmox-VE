# How to Configure Storage - Complete Beginner's Guide

## Question: How do I add storage in Proxmox VE?

### Answer: Follow these steps

---

## How to Add Local Storage

### Step 1: Go to Storage
1. **Datacenter** → **Storage** → **Add**

### Step 2: Select Type
Select **Directory** for local storage

### Step 3: Configure
- **Directory**: /var/lib/vz
- **Content**: ISO/Template, Root dir, Backups
- **Enable**: Yes

### Step 4: Save
Click **Create**

---

## How to Add NFS Storage

```bash
pvesm add nfs nfs-backup \
  --server 192.168.1.200 \
  --export /mnt/backup \
  --content backup,vztmpl,iso
```

---

## How to Add iSCSI Storage

```bash
pvesm add iscsi iscsi-main \
  --portal 192.168.1.250:3260 \
  --target iqn.name
```

---

## Keywords

#storage #how-to #beginner #nfs #iscsi

## Related Articles

- [[Storage]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
