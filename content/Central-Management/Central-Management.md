# Central Management Guide

## Proxmox VE GUI

Multiple nodes can be managed from single interface.

## Cluster Management

```
Datacenter
├── Node pve1
├── Node pve2
└── Node pve3
```

## API-based Management

Use single API endpoint to manage all nodes.

## Configuration Sync

```bash
# Sync config across nodes
rsync -av /etc/pve/ user@pve2:/etc/pve/
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
