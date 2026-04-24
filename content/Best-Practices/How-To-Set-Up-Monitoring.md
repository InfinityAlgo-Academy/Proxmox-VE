# How to Set Up Monitoring - Complete Guide

## Question: How do I monitor Proxmox properly?

### Answer: Use monitoring best practices

---

## How to Enable Basic Monitoring

### Step 1: Configure Email
1. **Datacenter** → **Notifications**
2. Set email server
3. Test notification

### Step 2: Check with CLI
```bash
# Node status
pvesh get /nodes/localhost/status

# Cluster status
pvecm status
```

---

## How to Set Up Advanced Monitoring

### Step 1: Install Prometheus
```bash
apt install prometheus-node-exporter
```

### Step 2: Install Grafana
```bash
apt install grafana
```

---

## How to Configure Alerts

### Step 1: Set Thresholds
```bash
# Disk alert
# Storage > 80%

# Memory alert  
# Usage > 90%

# CPU alert
# Load > 8 for 4 cores
```

---

## Keywords

#monitoring #grafana #prometheus #how-to #alerts

## Related Articles

- [[Best-Practices]]
- [[Monitoring-Tools]]
- [[Metrics]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
