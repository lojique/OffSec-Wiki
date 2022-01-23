---
description: >-
  In this lab, you will learn to exploit vulnerable applications and perform
  network pivoting.
---

# Black-box Penetration Test 1

## Lab Environment

In this lab environment, the user is going to get access to a Kali GUI instance. A web server can be accessed using the tools installed on Kali on **demo.ine.local**. However, there is another machine in the setup which is not accessible from the Kali machine but is accessible from the web server.

**Objective:** Compromise both machines to retrieve the flags!

## Tools

The best tools for this lab are:

* Metasploit Framework
* Nmap
* Curl

## My Solution

My IP: 192.123.133.2

Web Server IP: 192.123.133.3

```
# Nmap 7.91 scan initiated Sun Jan 23 05:22:44 2022 as: nmap -A -Pn -p- -oN scan.txt 192.39.180.0/24
Nmap scan report for linux (192.39.180.1)
Host is up (0.000065s latency).
Not shown: 65530 closed ports
PORT      STATE    SERVICE  VERSION
22/tcp    open     ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 37:36:c9:2c:34:4e:85:73:0b:b9:74:b9:a3:a3:20:e8 (RSA)
|   256 5d:62:6c:b1:4b:36:13:fb:11:2d:89:d0:8b:d1:55:bf (ECDSA)
|_  256 23:17:83:72:f2:94:bf:af:e6:cb:54:e4:aa:d3:92:59 (ED25519)
80/tcp    filtered http
443/tcp   filtered https
25555/tcp open     ssl/http Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| ssl-cert: Subject: commonName=example.com/organizationName=OrgName/stateOrProvinceName=Warwickshire/countryName=UK
| Not valid before: 2021-10-18T15:18:15
|_Not valid after:  2022-10-18T15:18:15
| tls-alpn: 
|_  http/1.1
29999/tcp open     http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:42:D9:23:C8:9B (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.06 ms linux (192.39.180.1)

Nmap scan report for target-1 (192.39.180.3)
Host is up (0.000026s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    nginx 1.14.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: nginx/1.14.0
|_http-title: V-CMS-Powered by V-CMS
3306/tcp open  mysql   MySQL (unauthorized)
MAC Address: 02:42:C0:27:B4:03 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.03 ms target-1 (192.39.180.3)

Nmap scan report for INE (192.39.180.2)
Host is up (0.000044s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE       VERSION
3389/tcp  open  ms-wbt-server xrdp
5910/tcp  open  vnc           VNC (protocol 3.8)
45654/tcp open  http          Apache Tomcat 8.5.57
|_http-title: Site doesn't have a title (text/html).
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Network Distance: 0 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jan 23 05:23:26 2022 -- 256 IP addresses (3 hosts up) scanned in 42.81 seconds
```

Missing IP: 192.198.175.2
