---
title: Lame
date: 2026-03-10
tags: hackthebox, easy, CVE
excerpt: VSFTPD v2.3.4 Backdoor Command Execution
---

### Nmap

```bash
21/tcp  open  ftp         syn-ack vsftpd 2.3.4
22/tcp  open  ssh         syn-ack OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```

### Exploitation

CVE-2007-2447,

```bash
Name: VSFTPD v2.3.4 Backdoor Command Execution
     Module: exploit/unix/ftp/vsftpd_234_backdoor
   Platform: Unix
       Arch: cmd
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2011-07-03
```