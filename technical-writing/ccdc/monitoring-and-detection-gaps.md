---
layout: technical-page
title: Monitoring and Detection Gaps
subtitle: Visibility failures that make compromise harder to spot during competition.
description: CCDC note on missing logs, disabled auditing, and weak detection coverage.
permalink: /technical-writing/ccdc/monitoring-and-detection-gaps/
---

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>

Hardening is incomplete if you cannot tell when a service is being attacked or abused. This page focuses on missing logs, disabled auditing, and other visibility gaps that make compromise harder to detect.

## Linux Logging

Common issues:

- `rsyslog` disabled
- `auditd` disabled
- Security-relevant events not being retained

Checks:

```bash
systemctl status rsyslog
systemctl status auditd
```

Fix:

```bash
sudo systemctl enable auditd
sudo systemctl start auditd
```

## Windows Logging

Common issues:

- Security auditing not enabled
- Login and privilege events not being captured

Open Event Viewer:

```text
eventvwr
```

Fix:

1. Open **Local Security Policy**.
2. Navigate to **Advanced Audit Policy Configuration**.
3. Enable login and privilege escalation auditing.

## Quick Review Checklist

When reviewing monitoring and detection, check for:

- system logs disabled or failing
- audit logging disabled
- authentication and privilege events not captured
- no clear place to review recent service activity

<p><a href="{{ '/technical-writing/ccdc/securing-a-machine/' | relative_url }}">Back to Securing a Machine</a></p>
