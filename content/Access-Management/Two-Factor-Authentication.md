# Two-Factor Authentication (2FA) in Proxmox VE

## Overview

Two-factor authentication adds an extra layer of security by requiring something you know (password) plus something you have (authentication device). Proxmox VE supports TOTP (Time-based One-Time Passwords) and YubiKey.

## Enabling 2FA

### Via Web UI

1. Navigate to **Datacenter → Users**
2. Select user → **2FA** tab
3. Click **Add**
4. Select authentication type:
   - **TOTP** (Authenticator app)
   - **YubiKey** (Hardware key)

## TOTP Configuration

### Required Steps

1. Install authenticator app:
   - Google Authenticator (iOS/Android)
   - Authy (iOS/Android)
   - Microsoft Authenticator (iOS/Android)
   - 1Password
   - Bitwarden

2. In Proxmox:
   ```
   User → 2FA → Add → TOTP
   ```

3. Scan QR code with authenticator app

4. Enter 6-digit code to enable

### Via CLI (Limited)

```bash
# View 2FA status for user
pveum user 2fa list admin@example.com

# Disable 2FA (requires password)
pveum user 2fa delete admin@example.com --method totp
```

## YubiKey Configuration

### Setup YubiKey

1. Connect YubiKey to USB port
2. In Proxmox:
   ```
   User → 2FA → Add → YubiKey
   ```
3. Touch YubiKey when prompted
4. Enable 2FA

### Using YubiKey

When logging in:
1. Enter username and password
2. Touch YubiKey button
3. Enter code from authenticator app (if using both)

## 2FA for Specific Users

### Enable for Administrators

```bash
# Always enable 2FA for admin users
# Via Web UI:
# Datacenter → Users → [admin] → 2FA → Enable TOTP
```

### Enable via PAM

For system users (local Linux accounts):
```bash
# Edit /etc/pve/user.cfg
# Add 2fa to user configuration
```

## 2FA Recovery

### Recovery Codes

Generate recovery codes before enabling 2FA:

1. User → 2FA → Add → TOTP
2. Click **Generate Recovery Codes**
3. Save codes securely (print/digital vault)
4. Use one code if authenticator unavailable

### Recovery Process

If locked out:
1. Use recovery code from saved list
2. Contact administrator for account recovery
3. Administrator can disable 2FA in user settings

## Disabling 2FA

```bash
# Via Web UI (as admin)
# Datacenter → Users → [User] → 2FA → Delete

# Only admin can disable another user's 2FA
```

## Security Considerations

### Backup Authenticator

- Set up 2FA on multiple devices
- Store recovery codes securely
- Use cloud-based authenticator with backup

### Hardware Keys

- YubiKey more secure than phone
- Works offline
- Physical possession required

## Enterprise 2FA with LDAP

Integration with LDAP for enterprise 2FA:

```bash
# Configure LDAP realm
pve realms add ldap --realm ldap.example.com
pve realms update ldap --base-dn "dc=example,dc=com"

# LDAP 2FA requires additional sync configuration
```

## Related Articles

- [[User Management]]
- [[Roles and Permissions]]
- [[Access Auditing]]
- [[Security Best Practices]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
