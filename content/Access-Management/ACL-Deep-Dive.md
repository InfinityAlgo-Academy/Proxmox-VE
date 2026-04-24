# Access Control Lists (ACL) Deep Dive

## Overview

ACLs (Access Control Lists) in Proxmox VE define who can access what resources and with what privileges. Understanding ACLs is crucial for proper security configuration.

## ACL Path Structure

### Path Hierarchy

```
/                           # Root - everything
/datacenter/                # Datacenter level
/storage/                  # Storage pools
/storage/local             # Local storage
/storage/local-lvm         # LVM storage
/vms/                      # All VMs
/vms/100                   # Specific VM
/vms/100/               # VM 100 and children
/vms/200                  # VM 200
/pool/                    # Resource pools
/pool/production           # Production pool
/access/                  # User management
/access/realm/            # Authentication realms
```

### Path Inheritance

Child paths inherit permissions from parents unless explicitly overridden.

```
/vms/100 has /vms/ permissions
/vms/ has / permissions
```

## ACL Operations

### Granting Permissions

```bash
# Single VM access
pveum acl modify /vms/100 --user admin@example.com --role PVEAdmin

# All VMs access
pveum acl modify /vms/ --user admin@example.com --role PVEAdmin

# Storage access
pveum acl modify /storage/local --user admin@example.com --role PVEAdmin

# Pool access
pveum acl modify /pool/production --group developers --role PVEUser
```

### Denying Access

```bash
# Explicit deny
pveum acl modify /vms/500 --user contractor@example.com --role NoAccess
```

### Modifying ACLs

```bash
# Update existing permission
pveum acl modify /vms/100 --user dev@example.com --role PVEPowerAdmin
```

### Removing ACLs

```bash
# Remove user permission
pveum acl delete /vms/100 --user admin@example.com

# Remove group permission
pveum acl delete /vms/100 --group developers
```

## Advanced ACL Patterns

### Environment-Based Access

```bash
# Production VMs (200-299)
for vm in {200..299}; do
  pveum acl modify /vms/$vm --group production --role PVEPowerAdmin
done

# Development VMs (100-199)
for vm in {100..199}; do
  pveum acl modify /vms/$vm --group developers --role PVEUser
done
```

### Role-Based Access Control

```bash
# Standard role assignments
pveum acl modify /vms/ --group "VM-Admins" --role PVEAdmin
pveum acl modify /vms/ --group "DevOps" --role PVEPowerAdmin
pveum acl modify /vms/ --group "Developers" --role PVEUser
pveum acl modify /vms/ --group "Auditors" --role PVEAudit
```

## ACL Inheritance Rules

### Inheritance Behavior

| Path | Granted Permission | Effective On |
|------|------------------|--------------|
| `/vms/100` | VM 100 | VM 100 only |
| `/vms/` | All VMs | All VMs |
| `/` | Everything | Everything |

### Override Examples

```bash
# Grant global access
pveum acl modify /vms/ --user user@example.com --role PVEUser

# Deny specific VM
pveum acl modify /vms/500 --user user@example.com --role NoAccess

# VM 500 still denied (explicit override takes precedence)
```

## ACL Best Practices

### Principle of Least Privilege

```bash
# Instead of:
pveum acl modify /vms/ --user dev@example.com --role PVEAdmin

# Use:
pveum acl modify /vms/ --user dev@example.com --role PVEUser
pveum acl modify /vms/201 --user dev@example.com --role PVEPowerAdmin
```

### Use Groups

```bash
# Create group for each access level
pveum group add vm-power-users
pveum group add vm-viewers

# Add users to groups
pveum group adduser vm-power-users user1@example.com
pveum group adduser vm-viewers user2@example.com

# Grant group permissions
pveum acl modify /vms/ --group vm-power-users --role PVEPowerAdmin
pveum acl modify /vms/ --group vm-viewers --role PVEAudit
```

## ACL Auditing

### List Permissions

```bash
# List all ACLs
pveum acl list

# List ACLs for specific VM
pveum acl list /vms/100

# List user permissions
pveum user list admin@example.com

# List group permissions
pveum list / --group developers
```

## Related Articles

- [[Roles and Permissions]]
- [[Groups and Groups Management]]
- [[User Management]]
- [[Access Auditing]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
