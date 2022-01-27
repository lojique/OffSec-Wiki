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

My IP: 192.104.6.2

![](<../../../../.gitbook/assets/image (13) (1) (1).png>)

## Footprinting & Scanning

### Nmap

```
root@INE:~# nmap -sV -sC -O -p1-10000 192.104.6.0/24
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-23 08:53 IST
Nmap scan report for linux (192.104.6.1)
Host is up (0.000035s latency).
Not shown: 9997 closed ports
PORT    STATE    SERVICE VERSION
22/tcp  open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 37:36:c9:2c:34:4e:85:73:0b:b9:74:b9:a3:a3:20:e8 (RSA)
|   256 5d:62:6c:b1:4b:36:13:fb:11:2d:89:d0:8b:d1:55:bf (ECDSA)
|_  256 23:17:83:72:f2:94:bf:af:e6:cb:54:e4:aa:d3:92:59 (ED25519)
80/tcp  filtered http
443/tcp filtered https
MAC Address: 02:42:E0:48:9F:C9 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for target-1 (192.104.6.3)
Host is up (0.000018s latency).
Not shown: 9998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    nginx 1.14.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: nginx/1.14.0
|_http-title: V-CMS-Powered by V-CMS
3306/tcp open  mysql   MySQL (unauthorized)
MAC Address: 02:42:C0:68:06:03 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop

Nmap scan report for INE (192.104.6.2)
Host is up (0.000023s latency).
Not shown: 9998 closed ports
PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server xrdp
5910/tcp open  vnc           VNC (protocol 3.8)
| vnc-info: 
|   Protocol version: 3.8
|   Security types: 
|     VeNCrypt (19)
|     VNC Authentication (2)
|   VeNCrypt auth subtypes: 
|     VNC auth, Anonymous TLS (258)
|_    Unknown security type (2)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Network Distance: 0 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 256 IP addresses (3 hosts up) scanned in 35.48 seconds
```

Web Server IP: 192.104.6.3

## Enumeration

Going to the website (demo.ine.local), it seems to be the way to get a foothold into the web server due to the fact that it's running v-cms v1.0

![](<../../../../.gitbook/assets/image (15) (1).png>)

We'll go ahead and fire up metasploit and search for available exploits

## Exploitation

The second option makes the most sense

![](<../../../../.gitbook/assets/image (33) (1) (1).png>)

We should read information about the exploit so we understand what's happening

```
info 1
```

Description:

> This module exploits a vulnerability found on V-CMS's inline image upload feature. The problem is due to the inline\_image\_upload.php file not checking the file type before saving it on the web server. This allows any malicious user to upload a script (such as PHP) without authentication, and then execute it with a GET request. The issue is fixed in 1.1 by checking the extension name. By default, 1.1 only allows jpg, jpeg, png, gif, bmp, but it is still possible to upload a PHP file as one of those extension names, which may still be leveraged in an attack.

Now we'll configure the options as such

![](<../../../../.gitbook/assets/image (34) (1).png>)

After typing run (or exploit), we get ourselves a meterpreter session!

![](<../../../../.gitbook/assets/image (7) (1) (1).png>)

Let's see what user we are

![](<../../../../.gitbook/assets/image (38) (1).png>)

Wow we're root!

### First Flag

We can try to retrieve the first flag. The first place I would check is root's home directory

![](<../../../../.gitbook/assets/image (24) (1).png>)

Now that we've gotten the first flag, we need to do the second part, which is find the other machine in the setup which is not accessible from the Kali machine but is accessible from the web server

## Pivoting

To check out the network configuration, we need to drop into a regular shell and upgrade the shell

```
shell
python -c 'import pty; pty.spawn("/bin/sh")'
bash
```

![](<../../../../.gitbook/assets/image (8) (1).png>)

Here is the network that we couldn't reach: `192.57.96.2`

![](<../../../../.gitbook/assets/image (17) (1) (1).png>)

To get back to our meterpreter shell, we'll enter in "exit" 3 times

### Autoroute

In order to compromise the other machine  which is on a different network, we need to use this machine  for routing traffic from a normally non-routable network

We want to leverage this newly discovered information and attack this additional network. Metasploit has an **autoroute** meterpreter script that will allow us to attack this second network through our first compromised machine

I'm using [this](https://www.offensive-security.com/metasploit-unleashed/pivoting/) as a guide

Commands:

```
run autoroute -s 192.57.96.0/24
run autoroute -p
```

![](<../../../../.gitbook/assets/image (36) (1) (1).png>)

### Port Scan

Now we need to determine what services are on this second network that we have discovered. We will use a basic TCP port scanner to look for ports

```
background
use auxiliary/scanner/portscan/tcp
show options
```

![](<../../../../.gitbook/assets/image (14) (1) (1).png>)

I don't expect anything wild so I'll just check for a few initial hosts

```
set ports 21, 22, 80, 8080
set rhosts 192.57.96.3-10
run
```

As we can see ports 21 and 22 are open

![](<../../../../.gitbook/assets/image (16) (1) (1).png>)

Now for port forwarding

### Port Forwarding

> The **portfwd** command from within the Meterpreter shell is most commonly used as a pivoting technique, allowing direct access to machines otherwise inaccessible from the attacking system. Running this command on a compromised host with access to both the attacker and destination network (or system), we can essentially forward TCP connections through this machine, effectively making it a pivot point. Much like the port forwarding technique used with an ssh connection, **portfwd** will relay TCP connections to and from the connected machines.

Source: [https://www.offensive-security.com/metasploit-unleashed/portfwd/](https://www.offensive-security.com/metasploit-unleashed/portfwd/)

```
sessions -i 3
portfwd add -l 4242 -p 21 -r 192.57.96.3
```

![](<../../../../.gitbook/assets/image (18) (1) (1).png>)

## Scanning Second Target

Now we need to run an nmap scan to gather more information about this new target

![](<../../../../.gitbook/assets/image (11) (1) (1).png>)

So we see here that this machine is running OpenSSH 6.6.1p1 (SSH) and vsftpd 2.0.8 or later (FTP)

## Exploiting Second Target

While searching for exploits, it becomes obvious we need the vsftpd exploit

![](<../../../../.gitbook/assets/image (12) (1) (1).png>)

Now we'll set up the payload and run it

![](<../../../../.gitbook/assets/image (21) (1) (1).png>)

Awesome, we got on the other machine!

### Second Flag

Now let's get that last flag

![](<../../../../.gitbook/assets/image (19) (1) (1).png>)

We have successfully exploited both  machines and retrieved both flags!
