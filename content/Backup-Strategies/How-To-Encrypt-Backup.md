# How to Encrypt Backup Data - Complete Guide

## Question: How do I secure sensitive backup data?

### Answer: Use encryption options available

---

## Why Encrypt Backups

### Benefits
- Protects from physical theft
- Secures data in transit
- Meets compliance requirements

---

## Option 1: PBS Encryption (Recommended)

### Step 1: Create Encryption Key
```bash
# Store securely - never share!
echo "your-secure-password" > /root/backup.key
chmod 600 /root/backup.key
```

### Step 2: Configure Encrypted Backup
1. **Datacenter** → **Backup** → **Add**
2. Enable **Encryption**
3. Enter password or select key file

### Step 3: CLI
```bash
vzdump --mode suspend \
  --vmid 100 \
  --storage pbs01 \
  --encryption-key-file /root/backup.key
```

---

## Option 2: GPG Encryption

### Step 1: Create Key
```bash
gpg --full-generate-key
# Select RSA 4096
# No expiration for backup key
```

### Step 2: Encrypt Backup File
```bash
# After backup completes
gpg -e -r your-key-id /var/lib/vz/dump/vzdump-*.tar.gz
```

---

## Option 3: VeraCrypt Container

### Step 1: Create Container
```bash
# Install veracrypt
apt install veracrypt
```

### Step 2: Create Encrypted Volume
```bash
veracrypt -c backup.vc
# Follow prompts
```

### Step 3: Mount and Use
```bash
veracrypt backup.vc /mnt/encrypted
cp /var/lib/vz/dump/* /mnt/encrypted/
veracrypt -d backup.vc
```

---

## Keywords

#encryption #backup-security #gpg #how-to #pbs

## Related Articles

- [[Backup]]
- [[Storage-Encryption]]
- [[Security]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
