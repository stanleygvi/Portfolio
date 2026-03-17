---
layout: technical-page
title: Service Whitelist by Machine Role
subtitle: Baseline processes and services to expect before deciding what to shut down.
description: CCDC service whitelist for common Linux and Windows machine roles.
permalink: /technical-writing/ccdc/services-whitelist/
---

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>

This page provides a baseline whitelist of services and processes expected on common machine roles. If a service appears outside the expected set for the host, it should be investigated.

This list is not exhaustive, but it covers the machine roles that come up often in CCDC-style environments.

## Linux Base System

These services appear on nearly every Linux system:

| Service / Process | Purpose |
| --- | --- |
| systemd | Init system / service manager |
| dbus-daemon | System messaging bus |
| sshd | Secure remote administration |
| cron / crond | Scheduled tasks |
| rsyslogd / systemd-journald | Logging |
| NetworkManager / systemd-networkd | Network management |
| agetty | Terminal login service |

## Linux Web Server

Typical stack: Nginx or Apache plus an application runtime.

| Service | Purpose |
| --- | --- |
| sshd | Remote administration |
| nginx | Web server |
| apache2 / httpd | Web server |
| php-fpm | PHP runtime |
| gunicorn / uwsgi | Python web applications |
| node | Node.js applications |
| cron | Scheduled jobs |
| rsyslog / journald | Logging |
| systemd | Service manager |

## Linux Database Server

Typical stack: MySQL, MariaDB, or PostgreSQL.

| Service | Purpose |
| --- | --- |
| sshd | Administration |
| mysqld / mariadbd | MySQL database |
| postgres | PostgreSQL database |
| cron | Backup jobs |
| rsyslog / journald | Logging |
| systemd | Service management |

## Linux File Server

Typical stack: Samba or NFS.

| Service | Purpose |
| --- | --- |
| sshd | Administration |
| smbd | Samba file sharing |
| nmbd | NetBIOS name service |
| nfs-server | NFS file sharing |
| rpcbind | NFS dependency |
| cron | Maintenance tasks |
| rsyslog | Logging |

## Linux Container Host

Typical stack: Docker or Kubernetes.

| Service | Purpose |
| --- | --- |
| docker / dockerd | Docker runtime |
| containerd | Container runtime |
| kubelet | Kubernetes node service |
| sshd | Administration |
| systemd | Service manager |
| rsyslog | Logging |

## Windows Base System

These processes appear on almost every Windows installation:

| Process | Purpose |
| --- | --- |
| System | Kernel system process |
| smss.exe | Session manager |
| csrss.exe | Client server runtime |
| wininit.exe | Windows initialization |
| services.exe | Service control manager |
| svchost.exe | Windows service host |
| lsass.exe | Local security authority |
| winlogon.exe | Login process |
| explorer.exe | Windows desktop shell |
| taskhostw.exe | Background tasks |

## Windows Domain Controller

Typical services for Active Directory:

| Process / Service | Purpose |
| --- | --- |
| lsass.exe | Authentication |
| dns.exe | Domain DNS |
| dfsr.exe | File replication |
| netlogon | Domain authentication |
| kdc | Kerberos authentication |
| svchost.exe | Service hosting |

## Windows Web Server (IIS)

Typical IIS-hosting processes:

| Process | Purpose |
| --- | --- |
| w3wp.exe | IIS worker process |
| inetinfo.exe | IIS service manager |
| svchost.exe | Windows services |
| lsass.exe | Authentication |
| services.exe | Service manager |

## Windows File Server

| Service | Purpose |
| --- | --- |
| LanmanServer | SMB file sharing |
| LanmanWorkstation | Network file access |
| svchost.exe | Windows services |
| lsass.exe | Authentication |
| services.exe | Service manager |

## Quick Verification Commands

Use these commands to compare the current host against the expected whitelist:

### Linux

```bash
ps aux
systemctl list-units --type=service --state=running
ss -tulpn
```

### Windows

```powershell
tasklist
Get-Service
netstat -ano
```

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>
