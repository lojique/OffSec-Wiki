---
description: >-
  In this lab, you will perform Scanning, Recon, and OS fingerprinting
  activities using Nmap scanner.
---

# Lab - Scanning and OS Fingerprinting

## Lab Environment

In this lab environment, the user is going to get access to a Kali GUI instance. There are a few machines on the same network running various services.

**Objective:** Perform scanning and OS fingerprinting using the Nmap tool and answer the following questions:

1. How many machines are there?
2. What ports are open on `pc1.ine.local` machine?
3. What OS is running on machine `pc1.ine.local` machine?
4. What services are running on `pc2.ine.local` machine?
5. What is the version of the FTP server running on one of the machines?
6. A caching server is also running on one of the machines. What is the domain name of that machine?
7. A NoSQL database and SQL database services are running on different machines. Can we use Nmap scripts to extract some information from those?

![](https://assets.ine.com/content/pta-labs/8\_scanning\_and\_os\_fingerprinting/0.png)

## Tools

The best tools for this lab are: - Nmap - Linux Terminal

## My Solution

Because we know that there are other machines on the same network as our Kali GUI instance, we need to know what our IP is in order to run a scan for the correct range

![](<../../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Pinging one of the local machines helps us make sure we scan the right network of IP range (in this case: 192.110.145.\*)

Next we run a simple nmap scan to see hosts on the network:

```bash
root@INE:~# nmap -sn 192.110.145.0/24
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-09 04:01 IST
Nmap scan report for linux (192.110.145.1)
Host is up (0.000050s latency).
MAC Address: 02:42:B9:34:7D:19 (Unknown)
Nmap scan report for target-1 (192.110.145.3)
Host is up (0.000015s latency).
MAC Address: 02:42:C0:6E:91:03 (Unknown)
Nmap scan report for target-2 (192.110.145.4)
Host is up (0.000013s latency).
MAC Address: 02:42:C0:6E:91:04 (Unknown)
Nmap scan report for target-3 (192.110.145.5)
Host is up (0.000013s latency).
MAC Address: 02:42:C0:6E:91:05 (Unknown)
Nmap scan report for target-4 (192.110.145.6)
Host is up (0.000036s latency).
MAC Address: 02:42:C0:6E:91:06 (Unknown)
Nmap scan report for INE (192.110.145.2)
Host is up.
Nmap done: 256 IP addresses (6 hosts up) scanned in 2.07 seconds
```

Next we'll go for a more accurate scan by scanning all 65535 ports and treating them all as open

```bash
root@INE:~# nmap -Pn -p- 192.110.145.1,2,3,4,5,6
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-09 04:05 IST
Nmap scan report for linux (192.110.145.1)
Host is up (0.000011s latency).
Not shown: 65530 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    filtered http
443/tcp   filtered https
25555/tcp open     unknown
29999/tcp open     bingbang
MAC Address: 02:42:B9:34:7D:19 (Unknown)

Nmap scan report for target-1 (192.110.145.3)
Host is up (0.000011s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
3306/tcp open  mysql
MAC Address: 02:42:C0:6E:91:03 (Unknown)

Nmap scan report for target-2 (192.110.145.4)
Host is up (0.000015s latency).
Not shown: 65534 closed ports
PORT      STATE SERVICE
27017/tcp open  mongod
MAC Address: 02:42:C0:6E:91:04 (Unknown)

Nmap scan report for target-3 (192.110.145.5)
Host is up (0.000012s latency).
Not shown: 65534 closed ports
PORT      STATE SERVICE
11211/tcp open  memcache
MAC Address: 02:42:C0:6E:91:05 (Unknown)

Nmap scan report for target-4 (192.110.145.6)
Host is up (0.000020s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
MAC Address: 02:42:C0:6E:91:06 (Unknown)

Nmap scan report for INE (192.110.145.2)
Host is up (0.0000060s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE
3389/tcp  open  ms-wbt-server
5910/tcp  open  cm
45654/tcp open  unknown

Nmap done: 6 IP addresses (6 hosts up) scanned in 8.55 seconds

```

It's beneficial to play around with nmap and try things you probably haven't to see how results might vary. I wanted to see how the results would show for a bunch of known ports and use the `--reason` tag

```bash
root@INE:~# nmap -Pn -p 21,22,80,443,53,139,445,25,3306,8080,8443 192.110.145.1,2,3,4,5,6 --reason
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-09 04:10 IST
Nmap scan report for linux (192.110.145.1)
Host is up, received arp-response (0.000013s latency).

PORT     STATE    SERVICE      REASON
21/tcp   closed   ftp          reset ttl 64
22/tcp   open     ssh          syn-ack ttl 64
25/tcp   closed   smtp         reset ttl 64
53/tcp   closed   domain       reset ttl 64
80/tcp   filtered http         no-response
139/tcp  closed   netbios-ssn  reset ttl 64
443/tcp  filtered https        no-response
445/tcp  closed   microsoft-ds reset ttl 64
3306/tcp closed   mysql        reset ttl 64
8080/tcp closed   http-proxy   reset ttl 64
8443/tcp closed   https-alt    reset ttl 64
MAC Address: 02:42:B9:34:7D:19 (Unknown)

Nmap scan report for target-1 (192.110.145.3)
Host is up, received arp-response (0.000033s latency).

PORT     STATE  SERVICE      REASON
21/tcp   closed ftp          reset ttl 64
22/tcp   closed ssh          reset ttl 64
25/tcp   closed smtp         reset ttl 64
53/tcp   closed domain       reset ttl 64
80/tcp   open   http         syn-ack ttl 64
139/tcp  closed netbios-ssn  reset ttl 64
443/tcp  open   https        syn-ack ttl 64
445/tcp  closed microsoft-ds reset ttl 64
3306/tcp open   mysql        syn-ack ttl 64
8080/tcp closed http-proxy   reset ttl 64
8443/tcp closed https-alt    reset ttl 64
MAC Address: 02:42:C0:6E:91:03 (Unknown)

Nmap scan report for target-2 (192.110.145.4)
Host is up, received arp-response (0.000017s latency).

PORT     STATE  SERVICE      REASON
21/tcp   closed ftp          reset ttl 64
22/tcp   closed ssh          reset ttl 64
25/tcp   closed smtp         reset ttl 64
53/tcp   closed domain       reset ttl 64
80/tcp   closed http         reset ttl 64
139/tcp  closed netbios-ssn  reset ttl 64
443/tcp  closed https        reset ttl 64
445/tcp  closed microsoft-ds reset ttl 64
3306/tcp closed mysql        reset ttl 64
8080/tcp closed http-proxy   reset ttl 64
8443/tcp closed https-alt    reset ttl 64
MAC Address: 02:42:C0:6E:91:04 (Unknown)

Nmap scan report for target-3 (192.110.145.5)
Host is up, received arp-response (0.000016s latency).

PORT     STATE  SERVICE      REASON
21/tcp   closed ftp          reset ttl 64
22/tcp   closed ssh          reset ttl 64
25/tcp   closed smtp         reset ttl 64
53/tcp   closed domain       reset ttl 64
80/tcp   closed http         reset ttl 64
139/tcp  closed netbios-ssn  reset ttl 64
443/tcp  closed https        reset ttl 64
445/tcp  closed microsoft-ds reset ttl 64
3306/tcp closed mysql        reset ttl 64
8080/tcp closed http-proxy   reset ttl 64
8443/tcp closed https-alt    reset ttl 64
MAC Address: 02:42:C0:6E:91:05 (Unknown)

Nmap scan report for target-4 (192.110.145.6)
Host is up, received arp-response (0.000015s latency).

PORT     STATE  SERVICE      REASON
21/tcp   open   ftp          syn-ack ttl 64
22/tcp   closed ssh          reset ttl 64
25/tcp   closed smtp         reset ttl 64
53/tcp   closed domain       reset ttl 64
80/tcp   closed http         reset ttl 64
139/tcp  closed netbios-ssn  reset ttl 64
443/tcp  closed https        reset ttl 64
445/tcp  closed microsoft-ds reset ttl 64
3306/tcp closed mysql        reset ttl 64
8080/tcp closed http-proxy   reset ttl 64
8443/tcp closed https-alt    reset ttl 64
MAC Address: 02:42:C0:6E:91:06 (Unknown)

Nmap scan report for INE (192.110.145.2)
Host is up, received user-set (0.000020s latency).

PORT     STATE  SERVICE      REASON
21/tcp   closed ftp          reset ttl 64
22/tcp   closed ssh          reset ttl 64
25/tcp   closed smtp         reset ttl 64
53/tcp   closed domain       reset ttl 64
80/tcp   closed http         reset ttl 64
139/tcp  closed netbios-ssn  reset ttl 64
443/tcp  closed https        reset ttl 64
445/tcp  closed microsoft-ds reset ttl 64
3306/tcp closed mysql        reset ttl 64
8080/tcp closed http-proxy   reset ttl 64
8443/tcp closed https-alt    reset ttl 64

Nmap done: 6 IP addresses (6 hosts up) scanned in 1.62 seconds
```



Now let's try to get more information about our targets using -sV for service versions and -O for OS detection

```bash
root@INE:~# nmap -Pn -sV -O -p- 192.110.145.1,2,3,4,5,6
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-09 04:13 IST
Nmap scan report for linux (192.110.145.1)
Host is up (0.000057s latency).
Not shown: 65530 closed ports
PORT      STATE    SERVICE  VERSION
22/tcp    open     ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    filtered http
443/tcp   filtered https
25555/tcp open     ssl/http Apache httpd 2.4.41 ((Ubuntu))
29999/tcp open     http     Apache httpd 2.4.41 ((Ubuntu))
MAC Address: 02:42:B9:34:7D:19 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for target-1 (192.110.145.3)
Host is up (0.000028s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 2.4.7
443/tcp  open  ssl/http Apache httpd 2.4.7 ((Ubuntu))
3306/tcp open  mysql    MySQL 5.5.47-0ubuntu0.14.04.1
MAC Address: 02:42:C0:6E:91:03 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop
Service Info: Host: localhost

Nmap scan report for target-2 (192.110.145.4)
Host is up (0.000033s latency).
Not shown: 65534 closed ports
PORT      STATE SERVICE VERSION
27017/tcp open  mongodb MongoDB 3.6.3
MAC Address: 02:42:C0:6E:91:04 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop

Nmap scan report for target-3 (192.110.145.5)
Host is up (0.000024s latency).
Not shown: 65534 closed ports
PORT      STATE SERVICE   VERSION
11211/tcp open  memcached Memcached 1.5.12 (uptime 2733 seconds)
MAC Address: 02:42:C0:6E:91:05 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop

Nmap scan report for target-4 (192.110.145.6)
Host is up (0.000077s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
MAC Address: 02:42:C0:6E:91:06 (Unknown)
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.3
Network Distance: 1 hop
Service Info: OS: Unix

Nmap scan report for INE (192.110.145.2)
Host is up (0.000055s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE       VERSION
3389/tcp  open  ms-wbt-server xrdp
5910/tcp  open  vnc           VNC (protocol 3.8)
45654/tcp open  http          Apache Tomcat 8.5.57
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Network Distance: 0 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 6 IP addresses (6 hosts up) scanned in 47.30 seconds
```



Now let's answer the questions

1. How many machines are there?
   * There are 6 machines
2. What ports are open on `pc1.ine.local` machine?

![](<../../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1).png>)

3\. What OS is running on machine `pc1.ine.local` machine?

![](<../../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1).png>)

4\. What services are running on `pc2.ine.local` machine?

![](<../../../../.gitbook/assets/image (8) (1) (1) (1) (1).png>)

5\. What is the version of the FTP server running on one of the machines?

![Although our most recent nmap scan answers this question, this particular does as well but with less info to search through](<../../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1).png>)

6\. A caching server is also running on one of the machines. What is the domain name of that machine?

* From the latest nmap scan result that was pasted, we have the caching server

```bash
11211/tcp open  memcached Memcached 1.5.12
```

* As for the domain, we know the machine is `target-3` with IP `192.110.145.6`

![](<../../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1).png>)

7\. A NoSQL database and SQL database services are running on different machines. Can we use Nmap scripts to extract some information from those?

### NoSQL

* I didn't really know what NoSQL was, but after researching MongoDB, I learned that it's classified as a NoSQL database program.  This [article ](https://www.mongodb.com/nosql-explained)explains it well.&#x20;
* NoSQL database service is running on target-2 so the syntax would be:

```bash
# there was a LOT of info, so I piped it to less
nmap -p 27017 --script mongodb-info target-2 | less
```

![](<../../../../.gitbook/assets/image (3) (1) (1) (1) (1).png>)

```bash
nmap -p 27017 --script mongodb-databases target-2
```

![](<../../../../.gitbook/assets/2022-01-08 20\_15\_55-INE Labs - Brave.png>)

Looks like there's no authentication needed

![](<../../../../.gitbook/assets/2022-01-08 20\_19\_45-INE Labs - Brave.png>)

### MySQL

Now we basically just repeat what we did for Mongodb against mysql

![](<../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/2022-01-08 20\_32\_04-INE Labs - Brave.png>)

![](<../../../../.gitbook/assets/2022-01-08 20\_35\_03-INE Labs - Brave.png>)

As you can see, we can gather a ton of information from nmap's NSE scripts

{% hint style="danger" %}
Using a wildcard, such as --script mysql-\* could get your requests blocked because a host can be configured to do that. Thse wildcards can generate a lot of noise and prevent you from making further requests
{% endhint %}

![](<../../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png>)

## Resources

* [https://nmap.org/nsedoc/categories/discovery.html](https://nmap.org/nsedoc/categories/discovery.html)
