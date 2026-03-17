---
layout: technical-page
title: Misconfigurations
subtitle: Unsafe settings, broad bindings, and information disclosure issues to look for.
description: CCDC note on reviewing service misconfigurations across common platforms.
permalink: /technical-writing/ccdc/misconfigurations/
---

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>

Many exposed-service issues come from unsafe settings, not unpatched software. This page focuses on service configuration problems such as unnecessary features, overly broad network exposure, and information disclosure.

## Web Servers

### Apache

Configuration files:

```bash
/etc/apache2/apache2.conf
/etc/httpd/conf/httpd.conf
```

Common issues:

- Directory listing enabled
- Version information exposed in headers or error pages

Fixes:

```text
Options -Indexes
ServerTokens Prod
ServerSignature Off
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

### Nginx

Configuration file:

```bash
/etc/nginx/nginx.conf
```

Common issues:

- `autoindex on;`
- `server_tokens on;`

Fixes:

```text
autoindex off;
server_tokens off;
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

### IIS

Open IIS Manager:

```text
inetmgr
```

Common issues:

- Directory browsing enabled
- Server header disclosure

Fixes:

- Open **Directory Browsing** and click **Disable**.
- Open **HTTP Response Headers** and remove or suppress the `Server` header if supported.

Restart IIS:

```powershell
iisreset
```

## Web Applications

Common locations:

Linux:

```bash
/var/www/
/var/www/html/
/srv/www/
```

Windows:

```text
C:\inetpub\wwwroot
```

Common issues:

- Weak file ownership or permissions in the web root
- Suspicious files such as dropped web shells
- Upload directories that allow script execution
- Unused extensions or plugins left installed

Fixes:

```bash
sudo chown -R www-data:www-data /var/www
sudo chmod -R 755 /var/www
find /var/www -type f
```

Investigate files such as:

```text
shell.php
cmd.php
upload.php
```

Disable execution in upload paths whenever possible and remove unused extensions or plugins.

## Databases

### MySQL / MariaDB

Configuration file:

```bash
/etc/mysql/mysql.conf.d/mysqld.cnf
```

Common issue:

- `bind-address = 0.0.0.0`

Fix:

```text
bind-address = 127.0.0.1
```

Restart MySQL:

```bash
sudo systemctl restart mysql
```

### PostgreSQL

Configuration files:

```bash
/etc/postgresql/*/main/postgresql.conf
/etc/postgresql/*/main/pg_hba.conf
```

Common issue:

- `listen_addresses = '*'`

Fix:

```text
listen_addresses = 'localhost'
```

Restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

## Containers

### Docker

Common issues:

- Unnecessary containers left running
- Unexpected published ports

Checks:

```bash
docker ps
docker inspect container_id
docker port container_id
```

Fixes:

```bash
docker stop container_id
docker rm container_id
```

Remove or reconfigure containers exposing unnecessary services or ports.

## Quick Review Checklist

When reviewing configurations, check for:

- services listening on all interfaces when they only need localhost or an internal IP
- directory listing enabled
- verbose banners or version disclosure
- upload paths that allow script execution
- unnecessary modules, plugins, containers, or connectors
- unnecessary exposed ports

If any of these appear, disable or restrict them wherever possible.

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>
