# How to Use Proxmox Backup Server (PBS) - Complete Guide

## Question: How do I set up and use Proxmox Backup Server?

### Answer: Follow these steps

---

## What is PBS?

Proxmox Backup Server provides:
- **Deduplication**: Saves storage by not duplicating data
- **Encryption**: Client-side encryption for security
- **Incremental**: Only backs up changes
- **Verification**: Data integrity checks

---

## How to Install PBS

### Option 1: Fresh Install

**Step 1: Download PBS ISO**
- Go to proxmox.com/downloads
- Download PBS ISO

**Step 2: Install**
- Boot from ISO
- Follow installation wizard

**Step 3: Access Web UI**
- Open browser: https://your-ip:8007

---

### Option 2: Install on Proxmox

```bash
# Add PBS repository
echo "deb http://download.proxmox.com/debian/pbs bullseye pve-no-subscription" > \
  /etc/apt/sources.list.d/pbs.list

# Update and install
apt update && apt install proxmox-backup-server -y

# Or on Proxmox VE
apt install proxmox-backup-client -y
```

---

## How to Configure PBS in Proxmox VE

### Step 1: Get PBS Fingerprint

In PBS web UI:
- **Configuration** → **Certificates**
- Copy fingerprint

### Step 2: Add PBS Storage

**Web UI Method:**
1. **Datacenter** → **Storage** → **Add** → **Proxmox Backup**
2. Fill in:
   - **Server**: 192.168.1.200
   - **Datastore**: main
   - **Username**: admin@pam
   - **Fingerprint**: (paste)

**CLI Method:**
```bash
pvesm add proxmox-backup-server pbs01 \
  --server 192.168.1.200 \
  --datastore main \
  --fingerprint YOUR-FINGERPRINT \
  --backup-username admin@pam
```

---

## How to Use PBS for Backup

### Select PBS Storage

1. **Datacenter** → **Backup** → **Add**
2. Choose **PBS storage** in target dropdown

### Configure as usual
- Normal backup settings apply

---

## How to Check PBS Status

```bash
# Check PBS
systemctl status proxmox-backup-server

# Check storage
pvesm status pbs01
```

---

## Keywords

#pbs #proxmox-backup-server #how-to #deduplication #encryption

## Related Articles

- [[Backup]]
- [[How-To-Backup-Storage]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
