# How to Monitor Proxmox - Complete Beginner's Guide

## Question: How do I monitor my Proxmox server?

### Answer: Multiple monitoring options

---

## Built-in Monitoring

### Using Web UI
Go to **Datacenter** → **Overview**

### Using CLI
```bash
# Node status
pvesh get /nodes/localhost/status

# Cluster status
pvecm status
```

---

## How to Enable Metrics

```bash
# Install Prometheus
apt install prometheus-node-exporter
```

---

## How to Set Up Alerts

```bash
# Configure email
# Datacenter → Notifications
```

---

## Keywords

#monitoring #how-to #beginner #metrics

## Related Articles

- [[Monitoring-Tools]]
- [[Metrics]]
- [[Proxmox-VE]]