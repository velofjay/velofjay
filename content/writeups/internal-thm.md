---
date: '2025-02-02T00:35:49+05:30'
draft: false
title: 'Internal: TryHackMe Walkthrough'
slug: internal-thm
tags: ["tryhackme", "wordpress", "phpmyadmin", "privilege-escalation"]
category: writeups
showtoc: true
summary: A walkthrough of the Internal room on TryHackMe, featuring WordPress enumeration, exploitation, and privilege escalation
description: Learn WordPress security assessment, directory enumeration, and privilege escalation techniques in this comprehensive walkthrough
---

# Internal: TryHackMe Walkthrough

## Initial Enumeration

### Nmap Scan Results
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
```

### Directory Enumeration
Using Gobuster:
```bash
/blog (Status: 301)
/wordpress (Status: 301)
/javascript (Status: 301)
/phpmyadmin (Status: 301)
/server-status (Status: 403)
```

## WordPress Assessment

### WPScan Results
```bash
wpscan --url 10.10.6.189/blog -e u,p,t
```

Key findings:
1. WordPress version 5.4.2 (Insecure)
2. XML-RPC enabled
3. WP-Cron enabled
4. User enumeration:
   - admin user identified

### Theme Versions
1. twentynineteen (v1.5)
   - Out of date (latest: 2.1)
2. twentyseventeen (v2.3)
   - Out of date (latest: 2.8)
3. twentytwenty (v1.2)
   - Out of date (latest: 1.8)

## Vulnerabilities Identified

1. **WordPress Core**
   - Outdated version 5.4.2
   - Known security issues

2. **XML-RPC**
   - Potential for brute force attacks
   - DDoS vulnerability

3. **Theme Vulnerabilities**
   - Multiple outdated themes
   - Potential for exploitation

## Exploitation Process

### WordPress Access
1. User enumeration
2. Password brute forcing
3. Admin panel access

### Privilege Escalation
1. WordPress plugin upload
2. Reverse shell execution
3. System enumeration

## Post Exploitation

### User Access
Successfully obtained user flag:
```bash
THM{int3rna1_fl4g_1}
```

## Best Practices

1. **WordPress Security**
   - Regular updates
   - Disable unnecessary features
   - Strong password policy

2. **Server Hardening**
   - Restrict directory access
   - Implement WAF
   - Regular security audits

3. **Access Control**
   - Limit admin access
   - Implement 2FA
   - Monitor login attempts

## Tools Used

1. **Enumeration**
   - Nmap
   - Gobuster
   - WPScan

2. **Exploitation**
   - WordPress exploits
   - Reverse shell scripts
   - Privilege escalation tools

3. **Post Exploitation**
   - System enumeration
   - Network analysis
   - Data extraction

## Additional Notes

1. **WordPress Configuration**
   - Remove version information
   - Disable XML-RPC if not needed
   - Configure security headers

2. **Server Security**
   - Update Apache configuration
   - Implement rate limiting
   - Enable logging

3. **Monitoring**
   - Set up intrusion detection
   - Monitor file changes
   - Log analysis

## References

1. [WordPress Security Guide](https://wordpress.org/support/article/hardening-wordpress/)
2. [WPScan Documentation](https://github.com/wpscanteam/wpscan)
3. [Apache Security Tips](https://httpd.apache.org/docs/2.4/misc/security_tips.html)
4. [TryHackMe - Internal](https://tryhackme.com/)
