---
title: Cluster
---

# Cluster Configuration

## Create Cluster

```bash
# Create cluster on node1
pvecm create production-cluster

# Add node2
pvecm add 192.168.1.101

# Add node3
pvecm add 192.168.1.102
```

## Cluster Commands

| Command | Description |
|---------|-------------|
| `pvecm status` | Show cluster status |
| `pvecm nodes` | List nodes |
| `pvecm expected` | Set expected votes |
| `pvecm delnode node1` | Remove node |

## High Availability

- [[High-Availability]]
- [[Cluster]]

## See Also

- [[High-Availability]]
- [[Cluster]]
- [[Installation]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
