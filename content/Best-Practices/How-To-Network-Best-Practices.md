# How to Configure Network Best Practices - Complete Guide

## Question: How do I set up network properly?

### Answer: Follow network best practices

---

## How to Implement Network Segmentation

### Step 1: Plan VLANs
| VLAN | Network | Purpose |
|------|---------|---------|
| 1 | 192.168.1.0/24 | Management |
| 10 | 192.168.10.0/24 | VM Network |
| 20 | 192.168.20.0/24 | Guest/Public |
| 30 | 192.168.30.0/24 | Storage |

### Step 2: Create VLAN Bridges
```bash
# Create VLAN 10
ip link add link vmbr0 name vmbr0.10 type vlan id 10
ip addr add 192.168.10.1/24 dev vmbr0.10
ip link set vmbr0.10 up
```

---

## How to Configure Network Bond

### Step 1: Create Bond
```bash
auto bond0
iface bond0 inet static
    bond-slaves eth0 eth1
    bond-mode 802.3ad
    bond-miimon 100
    bond-xmit-hash-policy layer2+3
```

---

## How to Configure DNS

### Step 1: Set Up DNS
```bash
# Using systemd-resolved
systemd-resolve --set-dns=1.1.1.1 --interface=enp1s0
systemd-resolve --set-dns=8.8.8.8 --interface=enp1s0
```

---

## Keywords

#network #vlan #bonding #segmentation #how-to

## Related Articles

- [[Best-Practices]]
- [[Networking]]
- [[Proxmox-VE]]