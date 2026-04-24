---
title: Open vSwitch Configuration Guide
---

# Open vSwitch Configuration Guide

## Overview

Open vSwitch (OVS) is a production-quality virtual switch supporting VLAN, QoS, sFlow, and OpenFlow protocols. Provides advanced networking features beyond Linux bridge.

## Install Open vSwitch

### Installation

```bash
# Install OVS
apt update
apt install openvswitch-switch -y

# Verify installation
ovs-vsctl show
```

### Start/Enable Service

```bash
# Start service
systemctl start openvswitch

# Enable on boot
systemctl enable openvswitch

# Check status
systemctl status openvswitch
```

## Basic Bridge Configuration

### Create Bridge

```bash
# Create bridge
ovs-vsctl add-br vmbr0

# Verify
ovs-vsctl list-br
```

### Add Ports

```bash
# Add physical interface
ovs-vsctl add-port vmbr0 eth0

# Add internal port
ovs-vsctl add-port vmbr0 veth0 -- set Interface veth0 type=internal

# Verify
ovs-vsctl list-ports vmbr0
```

### Configure IP

```bash
# Via Network Manager or:
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    ovs_type OVSBridge
    ovs_ports eth0
```

## VLAN Configuration

### Tagged VLANs

```bash
# Add VLAN port
ovs-vsctl add-port vmbr0 vlan100 tag=100 -- set Interface vlan100 type=inner

# Add trunk port
ovs-vsctl add-port vmbr0 trunk0 trunks=100,200,300 -- set Interface trunk0 type=trunk
```

### VLAN Ranges

```bash
# Native VLAN
ovs-vsctl add-port vmbr0 eth0 vlan_mode=native-untagged

# Trunk with native
ovs-vsctl add-port vmbr0 trunk0 trunks=100,200,300 vlan_mode=native-untagged
```

### VLAN Examples

| VLAN | Use Case |
|------|---------|
| 100 | Management |
| 200 | VM Network |
| 300 | Storage |

## QoS Configuration

### Rate Limiting

```bash
# Add QoS
ovs-vsctl -- set Port eth0 qos=@newqos \
  -- --id=@newqos create QoS type=linux-htb other-config:max-rate=1000000000

# Set minimum bandwidth
ovs-vsctl set qos newqos queues:1=@q1 \
  -- --id=@q1 create Queue other-config:min-rate=100000000
```

### Traffic Shaping

```bash
# Set priority
ovs-vsctl set Interface eth0 ingress_policing_rate=100000 \
  ingress_policing_burst=10000
```

## Bond/LAG Configuration

### LACP Bond

```bash
# Create bond
ovs-vsctl add-bond vmbr0 bond0 eth0 eth1 lacp=active

# Configure
ovs-vsctl set port bond0 bond_downdelay=0 bond_updelay=0
```

### Active-Backup

```bash
# Create active-backup bond
ovs-vsctl add-bond vmbr0 bond0 eth0 eth1 bond_mode=active-backup
```

### Balance-TLB

```bash
# Load balancing
ovs-vsctl add-bond vmbr0 bond0 eth0 eth1 bond_mode=balance-tlb lacp=passive
```

## VXLAN Tunneling

### Create VXLAN

```bash
# Create VXLAN
ovs-vsctl add-br vmbr0
ovs-vsctl add-port vmbr0 vxlan0 -- set interface vxlan0 type=vxlan \
  option:remote_ip=192.168.1.101 \
  option:key=1000 \
  option:dst_port=4789
```

### Multi-Site VXLAN

```bash
# Site A
ovs-vsctl add-port vmbr0 vxlan-siteA -- set interface vxlan-siteA type=vxlan \
  option:remote_ip=192.168.1.101 \
  option:key=100

# Site B
ovs-vsctl add-port vmbr0 vxlan-siteB -- set interface vxlan-siteB type=vxlan \
  option:remote_ip=192.168.1.100 \
  option:key=100
```

## Geneve Tunnel

### Create Geneve

```bash
# Create Geneve tunnel
ovs-vsctl add-br vmbr0
ovs-vsctl add-port vmbr0 geneve0 -- set interface geneve0 type=geneve \
  option:key=flow \
  option:remote_ip=192.168.1.101
```

### Geneve vs VXLAN

| Feature | VXLAN | Geneve |
|--------|------|-------|
| Header | Fixed | Variable |
| Tunnel ID | 24-bit | 24-bit |
| Flexible | Limited | Full |
| Use Case | L2 | Multi-cloud |

## Flow-Based Switching

### OpenFlow Rules

```bash
# Add flow
ovs-ofctl add-flow vmbr0 "priority=100,ip,tcp,nw_dst=192.168.1.10,actions=output:1"

# View flows
ovs-ofctl dump-flows vmbr0
```

### ACL-like Rules

```bash
# Block traffic
ovs-ofctl add-flow vmbr0 "priority=200,arp,actions=flood"

# Allow specific
ovs-ofctl add-flow vmbr0 "priority=150,in_port=1,ip,nw_src=10.0.0.0/24,actions=output:2"
```

## SSL/OpenFlow

### Configure SSL

```bash
# Generate certificates
ovs-pki init /var/lib/openvswitch/pki

# Create controller
ovs-pki req -u controller
ovs-pki sign controller

# Configure
ovs-vsctl set-controller vmbr0 ssl:192.168.1.200
```

### Connect Controller

```bash
# Add controller
ovs-vsctl set-controller vmbr0 tcp:192.168.1.200:6653
```

## Performance Tuning

### DPDK

```bash
# Enable DPDK
apt install openvswitch-switch-dpdk

# Configure DPDK
ovs-vsctl --set Open_vSwitch . other_config:dpdk-init=true

# Restart
systemctl restart openvswitch-switch
```

### NIC Offload

```bash
# Enable checksum offload
ovs-vsctl set Interface eth0 options:tx-check-sum-offload=true
```

## Monitoring

### Interface Stats

```bash
# Show interface stats
ovs-vsctl list interface eth0

# Port stats
ovs-ofctl dump-ports vmbr0

# Flow stats
ovs-ofctl dump-flows vmbr0
```

### Logging

```bash
# Enable logging
ovs-vsctl set Bridge vmbr0 flood-allow=true

# View log
tail -f /var/log/openvswitch/ovs-vswitch.log
```

## Troubleshooting

### Common Issues

```bash
# Bridge not created
ovs-vsctl add-br vmbr0

# Port not working
ovs-vsctl list-ports vmbr0

# Flows not working
ovs-ofctl dump-flows vmbr0
```

### Reset

```bash
# Delete all
ovs-vsctl del-br vmbr0

# Re-create
ovs-vsctl add-br vmbr0
```

## Quick Reference

### Common Commands

```bash
ovs-vsctl add-br <br>
ovs-vsctl add-port <br> <port>
ovs-vsctl del-port <br> <port>
ovs-vsctl list-br
ovs-vsctl show
```

## Keywords

#openvswitch #ovs #vxlan #geneve #network #vlan #qos #sdn #openflow #bonding

## Related Articles

- [[Advanced]]
- [[Networking]]
- [[Network-Config]]
- [[VLAN-Config]]
---

[[index|Back to Proxmox VE]]
