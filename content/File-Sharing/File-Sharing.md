---
title: File Sharing
---

# File Sharing

## SMB/CIFS

```bash
# Install Samba
apt install samba -y

# Configure
nano /etc/samba/smb.conf
```

## NFS

```bash
# Export NFS
echo "/share 192.168.1.0/24(rw,sync)" > /etc/exports
exportfs -a
```

## See Also

- [[index|Back to Proxmox VE]]