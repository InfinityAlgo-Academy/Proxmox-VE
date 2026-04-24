---
title: User Permissions
---

# User Permissions

## Create User

```bash
# Create user
pveum user add admin@example.com

# Set password
pveum user update admin@example.com --password newpass
```

## Permissions

```bash
# Grant permission
pveum acl modify /vms/ --user admin@example.com --role PVEAdmin
```

## See Also

- [[index|Back to Proxmox VE]]