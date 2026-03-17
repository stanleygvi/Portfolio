---
layout: technical-page
title: Weak Authentication and Access Control
subtitle: Reviewing who can log in and what they can do after they do.
description: CCDC note on authentication, access control, and overly broad administrative reach.
permalink: /technical-writing/ccdc/weak-authentication-and-access-control/
---

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>

Exposed services are often compromised through weak logins, default accounts, guest access, or overly broad administrative permissions. This page focuses on who can authenticate and what they can reach or change after authenticating.

## SSH

Configuration file:

```bash
/etc/ssh/sshd_config
```

Common issues:

- Root login enabled
- Password authentication enabled when keys would be safer
- Empty passwords permitted
- Too many login attempts allowed
- No restriction on which users may log in

Fixes:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no
MaxAuthTries 3
LoginGraceTime 30
AllowUsers adminuser
```

If password authentication must remain enabled, add brute-force protection such as **fail2ban**:

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

## Databases

### MySQL / MariaDB

Default or anonymous database accounts are a common foothold.

Run:

```bash
sudo mysql_secure_installation
```

This removes anonymous users, disables remote root login, removes the test database, and tightens authentication defaults.

### Microsoft SQL Server

If the database only serves a local application, remote connections may be unnecessarily broad.

Fix in SQL Server Management Studio:

1. Connect to the server.
2. Right-click the server and open **Properties**.
3. Select **Connections**.
4. Disable **Allow remote connections** if it is not required.

Restart the SQL Server service after changes.

## File Sharing Services

### Samba

Configuration file:

```bash
/etc/samba/smb.conf
```

Common issues:

- `guest ok = yes`
- `read only = no` on shares that do not need write access

Fixes:

```text
guest ok = no
read only = yes
```

Restart Samba:

```bash
sudo systemctl restart smbd
```

### NFS

Configuration file:

```bash
/etc/exports
```

Common issue:

- Shares exported to too many hosts or with unsafe privilege handling

Fix:

```text
/shared 192.168.1.10(rw,sync,root_squash)
```

Restart NFS:

```bash
sudo systemctl restart nfs-server
```

### Windows SMB

Check shares:

```powershell
Get-SmbShare
```

Fixes:

- Remove unnecessary shares.
- Open `secpol.msc`.
- Navigate to **Local Policies -> Security Options**.
- Disable **Accounts: Guest account status**.
- Restrict share access using NTFS permissions.

## Mail Systems

### Postfix

Configuration file:

```bash
/etc/postfix/main.cf
```

Common issue:

- Open relay behavior

Fix:

```text
smtpd_recipient_restrictions = permit_mynetworks, reject_unauth_destination
```

Restart Postfix:

```bash
sudo systemctl restart postfix
```

### Microsoft Exchange

Common issues:

- Overly broad administrative access
- Unused connectors left enabled
- Excessive mailbox permissions

Fixes:

- Restrict admin panel access.
- Disable unused connectors.
- Review mailbox permissions.

## Quick Review Checklist

When reviewing authentication and access control, check for:

- root or administrator logins allowed directly
- default, anonymous, or guest accounts still enabled
- public or overly broad write access
- unnecessary remote administration or database access
- no limits on login attempts
- no restriction on which users can access the service

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>
