# How to Exclude Files from Backup - Complete Guide

## Question: How do I exclude certain files or VMs from backup?

### Answer: Multiple exclusion options available

---

## How to Exclude VMs/Containers from Backup

### Method 1: Web UI

**Step 1: Edit Backup Job**
- Go to **Datacenter** → **Backup**
- Select backup job → **Edit**

**Step 2: Use Exclusion**
- Find **Exclude** option
- Add VM IDs to exclude
- Example: 100, 102

**Step 3: Save**
- Click **OK**

---

### Method 2: CLI Specific Exclusion

```bash
# Exclude specific VM
vzdump --mode snapshot --vmid 100,101,102 --exclude-vmid 100

# Exclude from "all"
vzdump --mode snapshot --vmid all --exclude-vmid 100
```

---

## How to Exclude Specific Files (Inside VM)

### Using Exclude File

Create file `/etc/vzdump-exclude.conf`:
```
/tmp/
/var/cache/
/var/tmp/
/swapfile
```

Backup automatically reads this file if present when using suspend mode.

---

## How to Exclude CD-ROM/ISO

```bash
# Remove ISO before backup
qm set 100 --ide2 none

# Or use exclude option
vzdump --mode snapshot --vmid 100 --exclude-media
```

---

## Pro Tips

1. **Exclude temp files** to reduce backup size
2. **Exclude logs** from production VMs

---

## Keywords

#exclude #how-to #beginner #optimization

## Related Articles

- [[Backup]]
- [[How-To-Configure-Backup]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
