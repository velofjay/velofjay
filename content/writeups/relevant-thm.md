---
date: '2025-02-02T00:35:49+05:30'
draft: false
title: 'Relevant: TryHackMe Walkthrough'
slug: relevant-thm
tags: ["tryhackme", "windows", "smb", "iis", "privilege-escalation"]
category: writeups
showtoc: true
summary: A walkthrough of the Relevant room on TryHackMe, featuring Windows enumeration, SMB exploitation, and IIS web server attacks
description: Learn Windows penetration testing techniques including SMB share enumeration, credential decoding, and web server exploitation
---

# Relevant: TryHackMe Walkthrough

## Initial Enumeration

### Nmap Scan Results
```bash
PORT     STATE SERVICE       VERSION
80/tcp   open  http         Microsoft IIS httpd 10.0
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2016 Standard Evaluation
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

### System Information
- OS: Windows Server 2016 Standard Evaluation 14393
- Computer Name: RELEVANT
- Workgroup: WORKGROUP

## SMB Enumeration

### Share Discovery
```bash
smbclient -L 10.10.232.115
```

### Share Access
```bash
smbclient //10.10.232.115/nt4wrksv
```

### Credential Discovery
Found encoded passwords in `passwords.txt`:
```
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```

### Decoded Credentials
```
Bob - !P@$$W0rD!123 
Bill - Juw4nnaM4n420696969!$$$
```

## Web Server Exploitation

### IIS Configuration
- Microsoft IIS 10.0
- TRACE method enabled (potential security risk)
- Web root accessible via SMB share

### Attack Vector
1. Generate ASPX reverse shell
2. Upload to accessible SMB share
3. Trigger execution via web server

## Privilege Escalation

### Process Enumeration
1. Identify running services
2. Check for vulnerable processes
3. Escalate privileges

## Best Practices

1. **SMB Security**
   - Disable SMB v1
   - Require message signing
   - Restrict anonymous access

2. **IIS Hardening**
   - Remove unnecessary HTTP methods
   - Implement proper access controls
   - Regular security updates

3. **Windows Security**
   - Apply security patches
   - Implement least privilege
   - Enable audit logging

## Tools Used

1. **Enumeration**
   - Nmap
   - enum4linux
   - smbclient

2. **Exploitation**
   - Metasploit Framework
   - Custom ASPX shell
   - Base64 decoder

3. **Post Exploitation**
   - Process Monitor
   - Windows commands
   - Privilege escalation scripts

## Additional Notes

1. **SMB Share Access**
   - Check permissions carefully
   - Look for sensitive files
   - Monitor file uploads

2. **Web Server Security**
   - Review HTTP methods
   - Check file permissions
   - Monitor uploads

3. **Credential Management**
   - Avoid storing encoded passwords
   - Use secure password storage
   - Regular password rotation

## Attack Path Summary

1. **Initial Access**
   - Enumerate SMB shares
   - Discover encoded credentials
   - Access sensitive shares

2. **Web Server Compromise**
   - Upload malicious ASPX
   - Execute reverse shell
   - Establish persistence

3. **Privilege Escalation**
   - Process enumeration
   - Exploit vulnerable service
   - Gain system access

## References

1. [Microsoft IIS Security Guide](https://docs.microsoft.com/en-us/iis/get-started/planning-your-iis-architecture/security-recommendations-for-iis)
2. [SMB Security Best Practices](https://docs.microsoft.com/en-us/windows-server/storage/file-server/security-best-practices)
3. [Windows Server 2016 Hardening](https://docs.microsoft.com/en-us/windows-server/security/security-and-assurance)
4. [TryHackMe - Relevant](https://tryhackme.com/)
