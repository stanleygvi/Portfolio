---
layout: technical-page
title: Out of Date Service Versions
subtitle: Checking services, runtimes, and images for patch gaps.
description: CCDC note on finding and updating outdated service versions.
permalink: /technical-writing/ccdc/out-of-date-service-versions/
---

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>

Outdated software is one of the most common vulnerabilities during competition hardening.

## Linux

Check service versions:

```bash
nginx -v
apache2 -v
mysql --version
psql --version
```

Update packages:

```bash
sudo apt update
sudo apt upgrade
```

Also review web application runtimes, plugins, and extensions. Update or remove anything that is no longer needed.

Check container images:

```bash
docker images
```

Replace old or untrusted images with current patched versions.

## Windows

Check installed programs:

```powershell
wmic product get name,version
```

Review versions for IIS-hosted applications, SQL Server, Exchange, and any third-party plugins. Run Windows Update if needed.

For Microsoft Exchange specifically, prioritize the latest available security updates because it is commonly targeted.

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>
