---
title: Netmon
date: 2026-03-10
tags: ctf, hackthebox, easy
excerpt: Paessler PRTG bandwidth monitor
---

### Nmap

```bash
PORT      STATE SERVICE      REASON  VERSION
21/tcp    open  ftp          syn-ack Microsoft ftpd
80/tcp    open  http         syn-ack Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
135/tcp   open  msrpc?       syn-ack
139/tcp   open  tcpwrapped   syn-ack
445/tcp   open  microsoft-ds syn-ack Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  tcpwrapped   syn-ack
47001/tcp open  tcpwrapped   syn-ack
49664/tcp open  tcpwrapped   syn-ack
49665/tcp open  tcpwrapped   syn-ack
49666/tcp open  msrpc        syn-ack Microsoft Windows RPC
49667/tcp open  msrpc        syn-ack Microsoft Windows RPC
49668/tcp open  tcpwrapped   syn-ack
49669/tcp open  unknown      syn-ack
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft: windows
```

### PORT 21 FTP

Allows anonymous login: `anonymous: anonymous`

I tried to read files and got `user.txt`

I left the FTP open in case I need to read other files.

### Port 80 Paessler PRTG bandwidth monitor

I tried the default credentials: `prtgadmin:prtgadmin` but it didn’t work.

I found backup program files at: `c:\programdata\paessler`

subl PRTG\ Configuration.old.bak

```xml
<dbpassword>
	      <!-- User: prtgadmin -->
	      PrTg@dmin2018
            </dbpassword>
            <dbtimeout>
              60
            </dbtimeout>
            <depdelay>
              0
```

I tried this password with user `prtgadmin` on the PRTG login page but it didn't work. Then I realized that this is from a 2018 backup, maybe the admin is lazy and re-used the dbpassword for the admin account and simply used the current date (2019).

So the current password is: `PrTg@dmin2019`