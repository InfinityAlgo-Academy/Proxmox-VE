# Roles and Permissions in Proxmox VE

## Overview

Proxmox VE uses a role-based access control (RBAC) system. Roles define what actions a user can perform, and permissions associate users or groups with specific roles on specific resources.

## Built-in Roles

### Standard Roles

| Role | Description | Privileges |
|------|-------------|------------|
| NoAccess | No access | None |
| PVEAudit | Read-only access | Auditor |
| PVEUser | Standard user | VM.PowerMgmt, VM.Monitor |
| PVEPowerAdmin | Power user | VM.PowerMgmt, VM.Monitor, VM.Console |
| PVEDisplayViewer | View-only console | VM.Console |
| PVEProxy | Proxmox Web Proxy | Sys.Audit, Sys.Console, Sys.Modify |
| PVESelfUser | Self-service user | Sys.Audit, PVESelfService |
| PVEAdmin | Full administrator | All privileges |

### VM-Specific Roles

| Role | Description |
|------|-------------|
| VM Admin | Full VM control (VM.\*) |
| VM Power User | Start, stop, restart (VM.PowerMgmt) |
| VM Monitor | View-only access (VM.Monitor) |
| VM Console | Console access only |
| VM Backup | Backup/restore access |

### Datacenter Roles

| Role | Description |
|------|-------------|
| SysAdmin | System administration |
| SysAudit | System read-only |
| SysModifier | System configuration |
| SysPowerMgmt | Power management |
| SysConsole | Console access |

## Creating Custom Roles

```bash
# Create custom role
pveum role add CustomRole --privelist "VM.PowerMgmt,VM.Monitor"

# Update role privileges
pveum role update CustomRole --addpriv "VM.Console"

# Delete role
pveum role delete CustomRole
```

## Permissions Management

### Granting Permissions

```bash
# Grant VM access
pveum acl modify /vms/100 --user admin@example.com --role PVEPowerAdmin

# Grant datastore access
pveum acl modify /storage/local --user admin@example.com --role PVEAudit

# Grant to group
pveum acl modify /vms/100 --group developers --role PVEUser

# Grant to API token
pveum acl modify /vms/100 --tokenid 'admin@example.com!mytoken' --role PVEUser
```

### Revoking Permissions

```bash
# Remove user permission
pveum acl delete /vms/100 --user admin@example.com

# Remove group permission
pveum acl delete /vms/100 --group developers
```

## Permission Path Structure

Permissions are applied hierarchically:

```
/
├── datacenter/
│   ├──storage/
│   │   └── local
│   ├── vms/
│   │   ├── 100
│   │   ├── 101
│   │   └── 102
│   └── pool/
│       └── pool名称
└──access/
    └──realm/
        └── realm名称
```

### Inheritance

- Permissions on `/vms/100` apply only to VM 100
- Permissions on `/vms/` apply to all VMs
- Permissions on `/` apply to everything

## Practical Examples

### Developer Access

```bash
# Create developer group
pveum group add developers

# Add users to group
pveum group adduser developers john@example.com
pveum group adduser developers jane@example.com

# Grant VM access to group
pveum acl modify /vms/ --group developers --role PVEUser
```

### Contractor Access (Limited)

```bash
# Create contractor role
pveum role add ContractorRole --privelist "VM.Monitor,VM.PowerMgmt"

# Grant limited access
pveum acl modify /vms/200 --user contractor@example.com --role ContractorRole

# Set expiration
pveum user update contractor@example.com --expire 2025-06-30
```

### Auditing Access

```bash
# List all ACLs
pveum acl list

# List ACLs for specific VM
pveum acl list /vms/100

# List user permissions
pveum user list admin@example.com
```

## Best Practices

1. **Use groups** - Manage permissions at group level
2. **Principle of least privilege** - Grant minimum necessary permissions
3. **Regular audits** - Review permissions quarterly
4. **Separate roles** - Differentiate between admin, power users, and regular users
5. **Document changes** - Keep log of permission changes

## Related Articles

- [[User Management]]
- [[Groups and Groups Management]]
- [[API Tokens]]
- [[Two-Factor Authentication]]