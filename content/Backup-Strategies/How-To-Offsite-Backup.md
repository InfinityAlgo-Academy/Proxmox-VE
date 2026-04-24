# How to Set Up Offsite Backups - Complete Guide

## Question: How do I keep backups offsite for disaster recovery?

### Answer: Multiple offsite options available

---

## Why Offsite Backups Matter

### Benefits
- Protection from site disasters (fire, flood, theft)
- Ransomware protection (isolated from network)
- Compliance requirements

---

## Option 1: rsync to Remote Server

### Step 1: Set Up Remote Server
```bash
# Install rsync on remote
apt install rsync

# Create backup user
useradd -m backupuser
mkdir /backup
chown backupuser:backupuser /backup
```

### Step 2: Configure SSH Key Auth
```bash
# Generate key
ssh-keygen -t rsa -b 4096

# Copy to remote
ssh-copy-id backupuser@192.168.1.200
```

### Step 3: Create Offsite Script
```bash
#!/bin/bash
# offsite-backup.sh

REMOTE="backupuser@192.168.1.200"
REMOTE_DIR="/backup"

# Sync all backups
rsync -avz --delete \
  /var/lib/vz/dump/ \
  $REMOTE:$REMOTE_DIR/

echo "$(date): Offsite backup complete" >> /var/log/offsite.log
```

### Step 4: Schedule
```bash
# Cron job
crontab -e

# Daily at 4 AM
0 4 * * * /root/offsite-backup.sh
```

---

## Option 2: Cloud Backup with rclone

### Step 1: Install rclone
```bash
curl https://rclone.org/install.sh | sudo bash
```

### Step 2: Configure Cloud
```bash
rclone config

# Follow prompts to add:
# - S3, Google Drive, Backblaze, etc.
```

### Step 3: Sync to Cloud
```bash
rclone sync /var/lib/vz/dump remote:bucket --progress
```

---

## Keywords

#offsite #cloud #rsync #rclone #how-to #disaster-recovery

## Related Articles

- [[Backup]]
- [[Disaster-Recovery]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
