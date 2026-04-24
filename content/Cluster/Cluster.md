# Proxmox VE Cluster

## What is a Proxmox Cluster?

A Proxmox cluster combines multiple Proxmox nodes into a single management domain with shared storage and High Availability (HA).

## Cluster Benefits

- **Centralized Management**: Single web UI for all nodes
- **High Availability**: Automatic VM/Container failover
- **Live Migration**: Move VMs without downtime
- **Shared Storage**: Access storage from any node
- **Distributed Resources**: Use resources across nodes

## Create Cluster

### On First Node
```bash
# Create cluster
pvecm create cluster-name

# Verify
pvecm status
```

### Add Node to Cluster
```bash
# Generate join token on first node
pvecm add node2-ip

# On second node, join
# Option 1: Web UI
# Node → Join Cluster → Enter info

# Option 2: CLI
pvecm add 192.168.1.101 -link0 192.168.1.101 --link1 192.168.1.102
```

### Cluster Network

For production, use dedicated network:
```bash
# Corosync (cluster traffic)
# Configure in /etc/corosync/corosync.conf
# Use dedicated NIC for cluster communication
```

## High Availability (HA)

### Enable HA
```bash
# Enable HA for VM
pct set 100 --ha 1
qm set 102 --ha 1

# Or via Web UI
# VM/Container → Hardware → High Availability
```

### HA Configuration
```bash
# Configure HA group
ha-manager group add homelab pve1 pve2 pve3

# Set VM priority
ha-manager crm add crm-config --group homelab --priority 100
```

### HA Behavior
- **Minimal**: Start when quorum available
- **Preferred**: Try to run on preferred node
- **Fence**: Use STONITH for reliability

## Live Migration

### Via Web UI
1. Select VM/Container
2. **Migrate** → Choose target node
3. **migrate**

### Via CLI
```bash
# Live migrate VM
qm migrate 100 pve2 --online

# Migrate container
pct migrate 100 pve2 --online

# Stopped VMs
qm migrate 100 pve2
```

## Fencing (STONITH)

### Configure Fencing
```bash
# Install fence-agents
apt install fence-agents -y

# Configure in Web UI
# Datacenter → HA → Fencing
```

## Quorum

### Understanding Quorum
```bash
# Check cluster quorum
pvecm status

# Expected votes: Number of cluster nodes
# Quorum: Expected/2 + 1
```

### Handle Split-Brain
- Use redundant network links
- Set proper fencing
- Configure watcher

## Distributed Cluster

### Multi-Site Cluster
```bash
# Use VXLAN/EVPN
# Or site-to-site VPN
# Configure corosync over VPN
```

## Cluster Commands

```bash
# Cluster status
pvecm status

# Nodes info
pvecm nodes

# Cluster logs
pvecm log

# Leave cluster
pvecm离开
# Or remove node
pvecm delnode nodename
```

## Monitoring

```bash
# Cluster resource list
crm status

# Check HA
ha-manager status
```

## Keywords

#cluster #ha #high-availability #migration #quorum #corosync #fencing

## Links

- [[Proxmox-VE]] - Main documentation
- [[Networking]] - Cluster networking
- [[Storage]] - Shared storage
- [[VM]] - VM migration
- [[Troubleshooting]] - Cluster issues
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
