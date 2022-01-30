# Black-box Penetration Test 2

## Scenario

* You have been engaged in a Black-box Penetration Test (`172.16.64.0/24` range). Your goal is to read the flag file on each machine. On some of them, you will be required to exploit a remote code execution vulnerability in order to read the flag.
* Some machines are exploitable instantly but some might require exploiting other ones first. Enumerate every compromised machine to identify valuable information, that will help you proceed further into the environment.
* If you are stuck on one of the machines, don't overthink and start pentesting another one.
* When you read the flag file, you can be sure that the machine was successfully compromised. But keep your eyes open - apart from the flag, other useful information may be present on the system.

> This is not a CTF! The flags' purpose is to help you identify if you fully compromised a machine or not.
>
> The solutions contain the shortest path to compromise each machine.You should follow the penetration testing process covered in its entirety!

## Goals

* Discover all the machines on the network
* Read all flag files (One per machine, stored on the filesystem or within a database)
* Obtain a reverse shell at least on 172.16.64.92

## What you will learn

* Taking advantage of DNS and virtual hosts
* Bypassing client-side access controls
* Abusing unrestricted file upload to achieve remote code execution

## Recommended tools

* Dirb
* Metasploit framework (recommended version 5)&#x20;
* Nmap&#x20;
* Sqlmap&#x20;
* BurpSuite&#x20;
* Text editor

## Stuff I need to write down

```bash
Nmap scan report for 172.16.64.81
Host is up, received user-set (0.047s latency).
Scanned at 2022-01-29 19:14:51 EST for 116s
Not shown: 59520 closed tcp ports (conn-refused), 6012 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:1e:bf:d0:44:0f:bc:c8:64:bd:ac:16:09:79:ca:a8 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCw55JDiDXMZSUODEktpM6HeFcANAVkGIM0XtYYmZ0KFkHfiy6hBFCvsii34LTg7lQY3dt1sKaH/bqBcVos0SU5rRCtjgr/3V9jXHN1u+hQLID4UssBTSwwikn2IoDdaay/J2TUb3Zl4rqjSOxjAmJ1VQaHUIuV9tZScOJDykZs9V02RZk48S7MToYv20MIkM7w7aLT7msEpHyXO5CX9GyUgGvMVGnv0mYGrxM7BZkdOiPLu53NzVCNXLfGqdu0po99z3QPFGr2MTnwcQ+FBgzmXeM+blQOFNwlC6JY9jQEy5F/m/6k/4bJtVL+gXVNmpTeq5YM1Dwh/KM8zsWeaaiV
|   256 df:60:fc:fc:db:4b:be:b6:3e:7a:4e:84:4c:a1:57:7d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAfWotA0wesTbPNILoHoYtzxahAiXgoKAQQ5NfdHBotI9Ce8MEK2+CQy8fSN/ZI7hyfPWA3Ed9d8I3hsA9gvlzs=
|   256 ce:8c:fe:bd:76:77:8e:bd:c9:b8:8e:dc:66:b8:80:38 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBIew3x3kc1O9hXM1Tkh+WU0+fTB4kyMjAK28aLNb6o4
80/tcp    open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
13306/tcp open  mysql   syn-ack MySQL 5.7.25-0ubuntu0.16.04.2
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.25-0ubuntu0.16.04.2
|   Thread ID: 17
|   Capabilities flags: 63487
|   Some Capabilities: FoundRows, InteractiveClient, IgnoreSigpipes, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, Speaks41ProtocolOld, LongColumnFlag, ODBCClient, LongPassword, SupportsCompression, Support41Auth, SupportsLoadDataLocal, SupportsTransactions, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: \x0Fd\x0Bc\x01*790h\\x0C0     \x135\x15|YJ
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.64.91
Host is up, received user-set (0.075s latency).
Scanned at 2022-01-29 19:14:51 EST for 116s
Not shown: 59561 closed tcp ports (conn-refused), 5972 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON  VERSION
80/tcp   open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
6379/tcp open  redis   syn-ack Redis key-value store

Nmap scan report for 172.16.64.92
Host is up, received user-set (0.10s latency).
Scanned at 2022-01-29 19:14:51 EST for 116s
Not shown: 59503 closed tcp ports (conn-refused), 6028 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f4:86:09:b3:d6:d1:ba:d0:28:65:33:b7:82:f7:a6:34 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmByIlC8cqwRm13XMsK7K9kUCDKY9ldboW/CfAXaPb6sArVM8bwkNPW4cB7DSb24ULsygMyhrkkPvJMWKyEjKlAzTa66/IzIQi6jfC9T/IRHLmTwowDdZtC4TZnNN5UklUXREqQ5iz4Xe9CEOWzUAMsLA9WdFoyYTZdcpw0RsoseRxIptBLNhB4kjsksVxbBqvnkitmGXIsaCCYqkl0DwBgP9wPP7JDEiOZZDbL9CtGKMTmmt37wsUdrJryaUeaACLTtAE//3ByxFCQ+JcUQ5+ay/y5pPZeje54n495mhCCCE56/oSGttt2Xv0r2oAhx0FnhVGM6rX45KfyTV6cGen
|   256 3b:d7:39:c3:4f:c4:71:a2:16:91:d1:8f:ac:04:a8:16 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEH1E1zluv269Xaic6ZWbvtA3GczG8vHsSyb4V3tjLm2qlvsEk3ZZgtiLIalEuRvMT0F/T1k/dSRM6AdkHN+mjs=
|   256 4f:43:ac:70:09:a6:36:c6:f5:b2:28:b8:b5:53:07:4c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINb3TYQv13wQfPcSfe+bZBkPcKGFvZYNvmqYp8vP94A2
53/tcp    open  domain  syn-ack dnsmasq 2.75
| dns-nsid: 
|_  bind.version: dnsmasq-2.75
80/tcp    open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Photon by HTML5 UP
63306/tcp open  mysql   syn-ack MySQL 5.7.25-0ubuntu0.16.04.2
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.25-0ubuntu0.16.04.2
|   Thread ID: 17
|   Capabilities flags: 63487
|   Some Capabilities: FoundRows, InteractiveClient, IgnoreSigpipes, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, Speaks41ProtocolOld, LongColumnFlag, ODBCClient, LongPassword, SupportsCompression, Support41Auth, SupportsLoadDataLocal, SupportsTransactions, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: fii|\x7F_C\x1Enz\x11IVL;Gu9+\x7F
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.64.166
Host is up, received user-set (0.041s latency).
Scanned at 2022-01-29 19:14:51 EST for 116s
Not shown: 59616 closed tcp ports (conn-refused), 5917 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON  VERSION
2222/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a6:1e:f8:c6:eb:32:0a:f6:29:c8:de:86:b7:4c:a0:d7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBg/PBPXZvP8p4mdUphuT5M891JReFZjDK0PNXZ4QY6lVMnGbCL/Ow+MqxXFIV/U+h8MpnTtO84Jtovoj6guFzxJayFa/GwuMwjS8124Xw38nFeUHZJsFxL/CQBVMdIVQZIkgQbSa30YI8ERGBSiHI3EOBpPp+BKG0W9AjXtTZE9RJiB1NrcHv7RlEnAHL5Qt/Kj613pioLVzPK7kpoqZk9AjKkzhfUVwfDHlbvRRMoa6HcLfZpF+lPuvPHlkd1sh1JwZFb088bLBnRh/IoRDOHT5LAVX8jdHekHBJ0jnovKk9ie7PeeOc9AMw/lH/ICDwXyhHBdi9SYFlblmXalYF
|   256 b9:94:56:c7:4d:63:ad:bd:2d:5e:26:43:75:78:07:6f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKhKsTT6L0sJ7ylyQb7rq4MyAEQF3EdZHNlh3sgx4J66bwrb26jc+WNuacfw8+1YPd2E0OF9NOUfX8KCisxFRuI=
|   256 d6:82:45:0a:51:4e:01:2d:6a:be:fa:cf:75:de:46:a0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILGXiL2CEU0iGi8INQq75FqhOlkNMPaUQFOvCab7iJej
8080/tcp open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Ucorpora Demo
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
