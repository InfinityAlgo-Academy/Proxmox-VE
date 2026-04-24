# How to Provide Beta Feedback - Complete Guide

## Question: How do I report beta issues?

### Answer: Feedback channels

---

## How to Report Bugs

### Step 1: Gather Information
```bash
# System info
pvesh get /version

# Logs
journalctl -xe > /tmp/pve-debug.log
```

### Step 2: Report to Proxmox
1. Forum: forum.proxmox.com
2. Bugtracker: bugzilla.proxmox.com
3. Include: logs, configs, steps to reproduce

---

## How to Test Beta Safely

### Step 1: Use Test Environment
```bash
# Clone production VM
qm clone 100 998 --name "test-clone"

# Test new feature
qm set 998 --beta-feature 1
```

---

## How to Share Feedback

### Step 1: Document Well
- Issue description
- Steps to reproduce
- Expected vs actual
- Workarounds found

---

## Keywords

#feedback #beta-testing #how-to #bugs

## Related Articles

- [[Beta-Features]]
- [[Proxmox-VE]]
- [[Community]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
