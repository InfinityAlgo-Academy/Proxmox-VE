---
title: Storage Encryption
---

# Storage Encryption

## LUKS Encryption

```bash
# Encrypt disk
cryptsetup luksFormat /dev/sdb

# Open encrypted disk
cryptsetup luksOpen /dev/sdb encrypted
```

## See Also

- [[index|Back to Proxmox VE]]