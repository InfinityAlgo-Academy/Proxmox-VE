# Networking Complete Guide

## Table of Contents

1. [Network Model](#network-model)
2. [Network Types](#network-types)
3. [Bridges](#bridges)
4. [Bonds (LAG)](#bonds-lag)
5. [VLANs](#vlans)
6. [Virtual Networks](#virtual-networks)
7. [Firewall](#firewall)
8. [Routing](#routing)
9. [DNS/DHCP](#dnsdhcp)
10. [Storage Networks](#storage-networks)
11. [Cluster Networks](#cluster-networks)
12. [Advanced Features](#advanced-features)
13. [Troubleshooting](#troubleshooting)

---

## Network Model

### Stack Architecture

```
┌──────────────────────────────────────────────────┐
│              Proxmox VE                        │
│  Web UI / CLI / API                         │
├──────────────────────────────────────────────────┤
│            OVS / Linux Bridge            │
├──────────────────────────────────────────────────┤
│           Bond (LACP/Active-Backup)       │
├──────────────────────────────────────────────────┤
│         VLAN (802.1Q Tagging)              │
├──────────────────────────────────────────────────┤
│             Physical NICs                   │
├──────────────────────────────────────────────────┤
│         Switch / Router                    │
└──────────────────────────────────────────────────┘
```

### Default Configuration

```
Internet
    │
    ▼
┌──────────────┐
│   Router    │
│  192.168.1.1│
└──────┬───────┘
       │ (trunk to Proxmox)
       ▼
┌──────────────┐
│    vmbr0    │ ← Management, VM traffic
│ 192.168.1.x│
└──────────────┘
```

---

## Network Types

### 1. Bridge Mode

```bash
# Linux Bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
```

### 2. Bond Mode

```bash
# Active-Backup (Failover)
auto bond0
iface bond0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bond-slaves eth0 eth1
    bond-mode active-backup
    bond-miimon 100
    bond-primary eth0

# LACP (802.3ad)
auto bond0
iface bond0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bond-slaves eth0 eth1
    bond-mode 802.3ad
    bond-miimon 100
    bond-xmit-hash-policy layer2+3
```

### 3. VLAN Mode

```bash
# VLAN Tagging
auto vmbr0.100
iface vmbr0.100 inet static
    address 192.168.100.100
    netmask 255.255.255.0
    vlan-raw-device vmbr0
```

---

## Bridges

### Create Bridge (Web UI)

1. **Node** → **Network** → **Create: Linux Bridge**
2. **Name**: `vmbr0`
3. **IPv4/CIDR**: `192.168.1.100/24`
4. **Gateway**: `192.168.1.1`
5. **Bridge Ports**: `eth0`
6. **VLAN Aware**: Yes/No

### Create Bridge (CLI)

```bash
# Create bridge
nano /etc/network/interfaces.d/vmbr0

auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports eth0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes

# Restart
systemctl networking restart
```

### Bridge Options

| Option | Description | Value |
|--------|-------------|-------|
| bridge-ports | Physical ports | eth0, eth1 |
| bridge-stp | Spanning Tree | on/off |
| bridge-fd | Forward delay | 0-30 |
| bridge-vlan-aware | VLAN support | yes/no |
| bridge-maxwait | Max wait | seconds |

### Multiple Bridges

```bash
# Management
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports eth0

# VM Network
auto vmbr1
iface vmbr1 inet manual
    bridge-ports eth1
    bridge-stp off

# Storage Network
auto vmbr2
iface vmbr2 inet manual
    bridge-ports eth2
    bridge-stp off
```

---

## Bonds (LAG)

### Bond Modes

| Mode | Name | Description | Failover |
|------|------|------------|----------|
| 0 | balance-rr | Round robin | No |
| 1 | active-backup | One active | Yes |
| 2 | balance-xor | XOR | Yes |
| 3 | broadcast | All ports | No |
| 4 | 802.3ad | LACP | Yes |
| 5 | balance-tlb | Load balance | Yes |
| 6 | balance-alb | Adaptive | Yes |

### Active-Backup (Recommended)

```bash
auto bond0
iface bond0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bond-slaves eth0 eth1
    bond-mode active-backup
    bond-miimon 100
    bond-primary eth0
    bond-updelay 200
    bond-downdelay 200
```

### LACP (802.3ad)

```bash
auto bond0
iface bond0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bond-slaves eth0 eth1
    bond-mode 802.3ad
    bond-miimon 100
    bond-lacp-rate 1
    bond-xmit-hash-policy layer2+3
    bond-min-links 1
```

### Bond Options

```bash
# Monitoring interval (ms)
bond-miimon 100

# Link up delay (ms)
bond-updelay 200

# Link down delay (ms)
bond-downdelay 200

# Hash policy
bond-xmit-hash-policy layer2  # MAC
bond-xmit-hash-policy layer2+3  # IP+MAC
bond-xmit-hash-policy encap2+3  # VLAN+IP+MAC
```

---

## VLANs

### Create VLAN

```bash
# Create VLAN interface
auto vmbr0.100
iface vmbr0.100 inet static
    address 192.168.100.100
    netmask 255.255.255.0
    vlan-raw-device vmbr0

# Multiple VLANs
auto vmbr0.100
auto vmbr0.200
auto vmbr0.300
```

### VLAN in VM/LXC

```bash
# LXC with VLAN
pct set 100 --net0 name=eth0,bridge=vmbr0,vlan=100,ip=dhcp

# VM with VLAN
qm set 100 --net0 virtio,bridge=vmbr0,vlan=100
```

### Native VLAN (Trunk)

```bash
# Set native VLAN
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports eth0
    bridge-stp off
    bridge-vlan-aware yes
    bridge-vids 100;200;300  # Tagged VLANs
```

### VLAN Configuration

| VLAN | Network | Purpose |
|------|--------|---------|
| 1 | 192.168.1.0/24 | Management |
| 10 | 192.168.10.0/24 | VMs |
| 20 | 192.168.20.0/24 | Guests |
| 30 | 192.168.30.0/24 | Storage |
| 40 | 192.168.40.0/24 | Backup |

---

## Virtual Networks

### Open vSwitch (OVS)

```bash
# Install OVS
apt install openvswitch-switch -y

# Create OVS bridge
ovs-vsctl add-br vmbr0

# Add ports
ovs-vsctl add-port vmbr0 eth0
ovs-vsctl add-port vmbr0 bond0

# Set VLAN
ovs-vsctl set port vmbr0 tag=100

# VLAN trunk
ovs-vsctl set port vmbr0 trunks=10,20,30
```

### VXLAN

```bash
# Create VXLAN
ovs-vsctl add-br vmbr0
ovs-vsctl add-port vmbr0 vxlan0 -- set interface vxlan0 type=vxlan option:remote_ip=192.168.1.101 option:key=1000
```

### Geneve

```bash
# Create Geneve tunnel
ovs-vsctl add-port vmbr0 geneve0 -- set interface geneve0 type=geneve option:key=flow option:remote_ip=192.168.1.101
```

---

## Firewall

### pfSense VM

```bash
# Create pfSense VM with multiple NICs
qm create 100 \
  --name pfsense \
  --memory 2048 \
  --cores 2 \
  --net0 e1000,bridge=vmbr0 \
  --net1 e1000,bridge=vmbr1 \
  --net2 e1000,bridge=vmbr2
  
# Install pfSense
# Configure via console
```

### NAT Configuration

```bash
# Enable NAT on host
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j MASQUERADE

# Persistent NAT
# /etc/network/if-up.d/nat
#!/bin/bash
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j MASQUERADE

chmod +x /etc/network/if-up.d/nat
```

### Port Forwarding

```bash
# Forward port to VM
iptables -t nat -A PREROUTING -p tcp -d 192.168.1.100 --dport 80 -j DNAT --to 192.168.1.200:80
```

### Firewall in Container

```bash
# Install ufw in container
pct exec 100 -- apt install ufw -y

# Configure
pct exec 100 -- ufw allow 22/tcp
pct exec 100 -- ufw allow 80/tcp
pct exec 100 -- ufw allow 443/tcp
pct exec 100 -- ufw enable
```

---

## Routing

### Static Routes

```bash
# Add route
ip route add 192.168.2.0/24 via 192.168.1.254

# Persistent
# /etc/network/interfaces
post-up ip route add 192.168.2.0/24 via 192.168.1.254
```

### Policy Routing

```bash
# Route via interface
ip route add default dev vmbr1 table 100
ip rule add from 192.168.10.0/24 table 100
```

### Reverse Path Filter

```bash
# Disable for VMs
sysctl -w net.ipv4.conf.all.rp_filter=0
sysctl -w net.ipv4.conf.default.rp_filter=0

# Persistent
# /etc/sysctl.d/99-rp-filter.conf
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```

---

## DNS/DHCP

### Dnsmasq

```bash
# Install
apt install dnsmasq -y

# Configure
# /etc/dnsmasq.conf
interface=vmbr0
bind-interfaces
dhcp-range=192.168.1.200,192.168.1.250,24h
dhcp-option=option:router,192.168.1.1
dhcp-option=option:dns-server,192.168.1.1
dhcp-option=option:domain-name,local

# Multiple ranges
dhcp-range=192.168.1.200,192.168.1.249,12h
dhcp-range=192.168.1.250,192.168.1.254,24h,vmbr1
```

### DHCP Reservations

```bash
# Static DHCP
# /etc/dnsmasq.conf
dhcp-host=BC:24:11:01:01:01,192.168.1.100
dhcp-host=BC:24:11:01:01:02,192.168.1.101

# Or by hostname
dhcp-host=bc:24:11:01:01:01,server,192.168.1.100
```

### DNS

```bash
# Custom DNS
address=/server.local/192.168.1.100
address=/nas.local/192.168.1.50
```

---

## Storage Networks

### Dedicated Storage NIC

```bash
# Separate NIC for storage
auto vmbr1
iface vmbr1 inet static
    address 10.0.1.100
    netmask 255.255.255.0
    bridge-ports eth1
    bridge-stp off
    bridge-fd 0

# Mount NFS over dedicated network
mount -t nfs 10.0.1.50:/data /mnt/nas
```

### iSCSI Network

```bash
# Create dedicated network
auto vmbr2
iface vmbr2 inet static
    address 10.0.2.100
    netmask 255.255.255.0
    bridge-ports eth2
    bridge-stp off

# Use for iSCSI
# Configure target on separate network
```

### Jumbo Frames

```bash
# Enable jumbo frames
mtu 9000

# Check
ip link show vmbr0 | grep mtu

# Requires switch support
```

---

## Cluster Networks

### Corosync

```bash
# Configure corosync network
# /etc/pve/corosync.conf
totem {
    interface {
        bindnetaddr: 192.168.1.100
        broadcast: 192.168.1.255
    }
}
```

### Dual Link

```bash
# Node 1
pvecm add 192.168.1.101 -link0 192.168.1.101 -link1 192.168.1.102

# Separate networks
# -link0: 192.168.1.x (cluster)
# -link1: 192.168.2.x (redundant)
```

---

## Advanced Features

### Network Filters

```bash
# ebtables for bridge filters
# Filter MAC addresses
ebtables -A FORWARD -s ! aa:bb:cc:dd:ee:ff -j DROP
```

### QoS

```bash
# Limit bandwidth (tc)
tc qdisc add dev vmbr0 root handle 1: htb default 10
tc class add dev vmbr0 parent 1: classid 1:10 htb rate 100mbit

# Apply to VM
iptables -A FORWARD -s 192.168.1.100 -j CLASSIFY --set-class 1:10
```

### Proxy ARP

```bash
# Enable proxy ARP
echo 1 > /proc/sys/net/ipv4/conf/all/proxy_arp
echo 1 > /proc/sys/net/ipv4/conf/vmbr0/proxy_arp
```

### Network Monitoring

```bash
# Check traffic
iftop

# Bandwidth
vnstat

# Detailed stats
ip -s link
```

---

## Troubleshooting

### Check Network

```bash
# View interfaces
ip addr

# Check routes
ip route

# Test connectivity
ping 8.8.8.8
ping google.com

# DNS
dig google.com
nslookup google.com
```

### Bridge Issues

```bash
# View bridges
brctl show

# View bridge details
bridge link

# Check bridge fdb
bridge fdb
```

### Bond Issues

```bash
# Check bond status
cat /proc/net/bonding/bond0

# View detailed
ip -d link show bond0
```

### VLAN Issues

```bash
# Check VLAN
ip -d link show vmbr0.100

# List VLAN devices
ls /proc/net/vlan
```

---

## Keywords

#networking #bridge #bond #vlan #lacp #firewall #nat #dns #dhcp #vxlan #openvswitch #qos #jumbo-frames #storage-network

## Links

- [[Proxmox-VE]] - Main documentation
- [[Installation]] - Initial network setup
- [[Storage]] - Storage networking
- [[Cluster]] - Cluster networking
- [[Troubleshooting]] - Network issues
---

[[index|Back to Proxmox VE]]
