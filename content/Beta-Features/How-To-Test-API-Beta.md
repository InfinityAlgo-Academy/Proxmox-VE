# How to Test API Beta Features - Complete Guide

## Question: How do I test new API features?

### Answer: API beta testing

---

## How to Enable API Beta

### Step 1: Enable
```bash
# Enable beta API
pvesh put /cluster/options --api-version 2
```

---

## How to Use New Endpoints

### Step 1: List Endpoints
```bash
# Get new endpoints
pvesh get /api2/json/

# Check version
pvesh get /version
```

---

## How to Test with Scripts

### Step 1: Test Scripts
```bash
#!/bin/bash
# test-new-api.sh

# Test new endpoint
curl -k -H "Authorization: PVEAPIToken=root@pam=test" \
  https://localhost:8006/api2/json/new/endpoint
```

---

## Keywords

#api #beta #testing #how-to

## Related Articles

- [[Beta-Features]]
- [[API]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
