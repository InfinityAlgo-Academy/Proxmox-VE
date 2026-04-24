# How to Configure iSCSI Storage - Complete Beginner's Guide

## Question: How do I set up iSCSI storage?

### Answer: Follow these steps

---

## How to Add iSCSI Storage

### Step 1: Add in Proxmox
1. **Datacenter** → **Storage** → **Add** → **iSCSI**
2. Portal: 192.168.1.250:3260
3. Target: iqn.name
4. Enable

### Step 2: CLI Method
```bash
pvesm add iscsi iscsi-storage \
  --portal 192.168.1.250:3260 \
  --target iqn.your.target
```

---

## Keywords

#iscsi #storage #how-to #beginner

## Related Articles

- [[iSCSI]]
- [[Storage]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
