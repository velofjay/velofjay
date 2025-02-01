---
date: '2025-02-02T00:07:56+05:30'
draft: false
title: 'Steel Mountain: TryHackMe Walkthrough'
slug: steel-mountain-thm
tags: ["tryhackme", "windows", "metasploit", "powershell", "privesc"]
category: writeups
showtoc: true
summary: A walkthrough of the Steel Mountain room on TryHackMe, focusing on Windows enumeration, web server exploitation, and privilege escalation
description: Complete guide to exploiting a Windows machine through vulnerable web server, using Metasploit for initial access and PowerUp for privilege escalation
---

# Steel Mountain: TryHackMe Walkthrough

## Introduction

Steel Mountain is a Windows machine that requires enumeration, exploitation of a vulnerable web server, and privilege escalation using PowerShell techniques.

## Initial Reconnaissance

### Web Server Enumeration
- Primary web server running on port 80
- Secondary HFS (HTTP File Server) on port 8080
- Employee of the month: Bill Harper
  - Found through image metadata/location
  - Alternatively discoverable through reverse image search

## Initial Access

### Vulnerable Service
- HFS (Rejetto HTTP File Server) 2.3 running on port 8080
- Known vulnerability: CVE-2014-6287
- Exploit available on [Exploit-DB](https://www.exploit-db.com/exploits/39161)

### Exploitation Process
1. Identify HFS version
2. Locate appropriate exploit in Metasploit
3. Execute Remote Code Execution (RCE) vulnerability
4. Gain initial shell access

## Privilege Escalation

### PowerUp Enumeration
- Using [PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) for Windows privilege escalation enumeration
- Script identifies common Windows security misconfigurations
- Searches for privilege escalation vectors and abnormalities

### Key Areas to Check
1. Service Misconfigurations
2. Unquoted Service Paths
3. Weak Registry Permissions
4. Weak File Permissions

## Best Practices

1. **Initial Enumeration**
   - Port scanning
   - Service identification
   - Version detection

2. **Exploitation**
   - Verify vulnerability exists
   - Test exploit in isolated environment
   - Document changes made

3. **Post-Exploitation**
   - Maintain access
   - Gather system information
   - Document findings

## Tools Used

1. **Metasploit Framework**
   - Initial exploitation
   - Payload delivery
   - Session management

2. **PowerShell Scripts**
   - PowerUp for enumeration
   - Privilege escalation checks
   - System analysis

3. **Enumeration Tools**
   - Nmap for port scanning
   - Web browsers for reconnaissance
   - Version identification tools

## Security Implications

1. **Web Server Security**
   - Keep services updated
   - Remove unnecessary services
   - Implement proper access controls

2. **Windows Hardening**
   - Regular security updates
   - Service permission audits
   - User privilege reviews

## References

1. [HFS (HTTP File Server) Documentation](http://www.rejetto.com/hfs/)
2. [PowerSploit GitHub Repository](https://github.com/PowerShellMafia/PowerSploit)
3. [Windows Privilege Escalation Guide](https://www.fuzzysecurity.com/tutorials/16.html)
4. [CVE-2014-6287 Details](https://nvd.nist.gov/vuln/detail/CVE-2014-6287)
