---
title: Snapshot Management
---

# Snapshot Management

## Create Snapshot

```bash
# VM snapshot
qm snapshot 100 backup1 --description "pre-update"

# Container snapshot
pct snapshot 200 backup1
```

## List Snapshots

```bash
# List VM snapshots
qm snapshot list 100

# List CT snapshots
pct snapshot list 200
```

## Delete Snapshot

```bash
qm snapshot delete 100 backup1
pct snapshot delete 200 backup1
```

## See Also

- [[index|Back to Proxmox VE]]