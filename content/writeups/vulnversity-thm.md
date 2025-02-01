---
date: '2025-02-01T23:17:39+05:30'
draft: false
title: 'Vulnversity: TryHackMe Walkthrough'
slug: vulnversity-thm
tags: ["tryhackme", "ctf", "pentesting", "web", "privesc"]
category: writeups
showtoc: true
summary: A detailed walkthrough of the Vulnversity room on TryHackMe, covering reconnaissance, web server exploitation, and privilege escalation
description: Complete guide for TryHackMe's Vulnversity challenge demonstrating the process of enumerating and exploiting a vulnerable web server, followed by privilege escalation using SUID binaries
---

# Vulnversity: TryHackMe Walkthrough

## Reconnaissance

Initial enumeration was performed using nmap to identify open ports and running services:

```bash
nmap -sV <machine-ip>
```

Key findings:
- Web server running
- Multiple open ports identified
- Service versions enumerated for potential vulnerabilities

## Web Directory Enumeration

Used GoBuster to discover hidden directories on the web server:

```bash
gobuster dir -u http://<machine-ip> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

This revealed several interesting directories that weren't visible through normal browsing.

## Web Server Compromise

### Initial Access
1. Located upload functionality in the web application
2. Tested various file upload bypass techniques
3. Successfully uploaded a reverse shell payload
4. Gained initial foothold on the system

### Reverse Shell
Successfully established a reverse shell connection to the target system, providing command execution capabilities.

## Privilege Escalation

### SUID Binary Enumeration

Located SUID binaries using the following command:
```bash
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

Notable SUID binaries found:
```
-rwsr-xr-x 1 root root 32944 May 16  2017 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 49584 May 16  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 136808 Jul  4  2017 /usr/bin/sudo
-rwsr-xr-x 1 root root 659856 Feb 13  2019 /bin/systemctl
-rwsr-xr-x 1 root root 40128 May 16  2017 /bin/su
-rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
```

### Root Access
- Identified vulnerable SUID binary
- Exploited binary permissions to escalate privileges
- Successfully obtained root access

## Key Learnings

1. Web Application Security:
   - File upload vulnerabilities
   - Extension filtering bypass techniques
   - Reverse shell deployment

2. System Enumeration:
   - Service version identification
   - Directory discovery
   - SUID binary enumeration

3. Privilege Escalation:
   - Understanding SUID permissions
   - Binary exploitation
   - Linux privilege escalation techniques

## Tools Used

1. Reconnaissance:
   - nmap
   - GoBuster
   - dirb wordlists

2. Exploitation:
   - PHP reverse shell
   - netcat listener
   - Linux enumeration scripts

## References

1. [GTFOBins](https://gtfobins.github.io/)
2. [PayloadsAllTheThings - File Upload Bypass](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
3. [TryHackMe - Vulnversity Room](https://tryhackme.com/room/vulnversity)
