# How to Set Up User Permissions - Complete Guide

## Question: How do I manage users properly?

### Answer: Follow user management best practices

---

## How to Create Admin User

### Step 1: Create User
```bash
useradd -m -s /bin/bash admin
usermod -aG sudo admin
passwd admin
```

---

## How to Assign Permissions

### Step 1: Set VM Permissions
```bash
# Grant VM access
pveum acl modify /vms/100 --user admin@example.com --role PVEPowerAdmin

# Grant storage access
pveum acl modify /storage/local --user admin@example.com --role PVEAudit
```

---

## How to Create Groups

### Step 1: Create Group
```bash
pveum group add developers
pveum group adduser developers user@example.com
```

---

## Keywords

#users #permissions #acl #how-to #management

## Related Articles

- [[Best-Practices]]
- [[Access-Management]]
- [[Proxmox-VE]]