---
layout: technical-page
title: Securing a Machine
subtitle: CCDC hardening workflow with linked review notes for exposed services.
description: Practical CCDC machine-hardening checklist with supporting service review notes.
permalink: /technical-writing/ccdc/securing-a-machine/
---

<p><a href="{{ '/technical-writing/technical-experiences/ccdc-notes-in-progress/' | relative_url }}">Back to CCDC Notes</a></p>

This is a guide to securing a machine based on my experience in CCDC competitions.

## 1. Backup

This matters because if Red Team gets in and encrypts or deletes important data, you still have a clean snapshot to recover from.

### Linux

#### Backup web data

```bash
sudo tar -czf /root/web_backup.tar.gz /var/www
```

#### Backup configs

```bash
sudo tar -czf /root/config_backup.tar.gz /etc
```

#### Backup databases

```bash
mysqldump -u root -p --all-databases > db_backup.sql
```

### Windows

#### Backup important directories

```powershell
robocopy C:\inetpub C:\backup /E
```

## 2. Change Default Credentials

Change defaults early because once Red Team finds a weak or default credential, they can often move quickly across the environment.

This includes:

- root or administrator passwords
- user passwords
- service account passwords
- database credentials
- SSH keys

### Linux

List users:

```bash
getent passwd
```

Change a password:

```bash
sudo passwd USERNAME
```

### Windows

List users:

```powershell
net user
Get-LocalUser
```

Change a password:

```powershell
net user USERNAME NEW_PASSWORD
```

## 3. Terminate Unnecessary Services

Once the obvious front door is closed, reduce attack surface by shutting down services the machine does not actually need for scoring.

Start by identifying the host's intended role from the team packet, then compare running services against the expected role baseline:

- [Service Whitelist by Machine Role]({{ '/technical-writing/ccdc/services-whitelist/' | relative_url }})

After that, close ports and disable services that are not required.

## 4. Inspect Active Services for Vulnerabilities

After unnecessary services are disabled, the remaining exposed services become the primary attack surface. I break service review into four buckets:

- [Out of Date Service Versions]({{ '/technical-writing/ccdc/out-of-date-service-versions/' | relative_url }})
- [Misconfigurations]({{ '/technical-writing/ccdc/misconfigurations/' | relative_url }})
- [Weak Authentication and Access Control]({{ '/technical-writing/ccdc/weak-authentication-and-access-control/' | relative_url }})
- [Monitoring and Detection Gaps]({{ '/technical-writing/ccdc/monitoring-and-detection-gaps/' | relative_url }})

Start with internet-facing services first, then move inward to internal-only services.

## 5. Look Through the System for Remaining Vulnerabilities

Once exposed services are reviewed, continue with broader host checks:

- turn on Windows Defender where applicable
- review autostarting applications and scheduled tasks
- look for suspicious files, scripts, and persistence points
- remove software or tools that should not be present

<p><a href="{{ '/technical-writing/technical-experiences/ccdc-notes-in-progress/' | relative_url }}">Back to CCDC Notes</a></p>
