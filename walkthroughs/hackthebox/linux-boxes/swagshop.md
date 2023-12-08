---
description: 'IP: 10.10.10.140'
---

# SwagShop

## NMAP

{% code overflow="wrap" %}
```bash
# Nmap 7.93 scan initiated Tue Oct 18 21:21:09 2022 as: nmap -vvv -p 22,80 -Pn -sCV -oN nmap.txt 10.10.10.140
Nmap scan report for swagshop.htb (10.10.10.140)
Host is up, received user-set (0.022s latency).
Scanned at 2022-10-18 21:21:09 EDT for 8s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 b6552bd24e8fa3817261379a12f624ec (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCgTCefp89MPJm2oaJqietdslSBur+eCMVQRW19iUL2DQSdZrIctssf/ws4HWN9DuXWB1p7OR9GWQhjeFv+xdb8OLy6EQ72zQOk+cNU9ANi72FZIkpD5A5vHUyhhUSUcnn6hwWMWW4dp6BFVxczAiutSWBVIm2YLmcqwOEOJhfXLVvsVqu8KUmybJQWFaJIeLVHzVgrF1623ekDXMwT7Ktq49RkmqGGE+e4pRy5pWlL2BPVcrSv9nMRDkJTXuoGQ53CRcp9VVi2V7flxTd6547oSPck1N+71Xj/x17sMBDNfwik/Wj3YLjHImAlHNZtSKVUT9Ifqwm973YRV9qtqtGT
|   256 2e30007a92f0893059c17756ad51c0ba (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEG18M3bq7HSiI8XlKW9ptWiwOvrIlftuWzPEmynfU6LN26hP/qMJModcHS+idmLoRmZnC5Og9sj5THIf0ZtxPY=
|   256 4c50d5f270c5fdc4b2f0bc4220326434 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINmmpsnVsVEZ9KB16eRdxpe75vnX8B/AZMmhrN2i4ES7
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 88733EE53676A47FC354A61C32516E82
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Home page
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Oct 18 21:21:17 2022 -- 1 IP address (1 host up) scanned in 8.35 seconds
```
{% endcode %}

> SERVICE VERSIONS
>
> SSH: OpenSSH 7.2p2 Ubuntu 4ubuntu2.8
>
> HTTP: Apache 2.4.18

## HTTP

<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

### webfuzz

```bash
dirsearch -u http://10.10.10.140 -w /usr/share/dirb/wordlists/common.txt -t 150 -x 404
```

<figure><img src="../../../.gitbook/assets/image (582).png" alt=""><figcaption></figcaption></figure>

Ran dirsearch and checked out the first item which was `http://10.10.10.140/app/`

From there, I discovered a local.xml file:

<figure><img src="../../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

Its contents reveal something interesting

<figure><img src="../../../.gitbook/assets/image (492).png" alt=""><figcaption><p>local.xml file</p></figcaption></figure>

Based on the content shown in the image above, it would seem likely these credentials would work for mysql. However, because mysql wasn't shown in our  nmap scan, perhaps this is running internally.

```
root
fMVWh7bDHpgZkyfqQXreTjU9
```

I decided to revisit my webfuzzing because from prior experience, sometimes you need to fuzz beyond the initial given target.

<figure><img src="../../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

Now we visit the URL

<figure><img src="../../../.gitbook/assets/image (469).png" alt=""><figcaption></figcaption></figure>

Tried the classic `admin:admin`, as well as the mysql username and password (in case of password reuse), and some other random credentials. Unfortunately none of these worked.

### searchsploit

<figure><img src="../../../.gitbook/assets/image (550).png" alt=""><figcaption></figcaption></figure>

I used searchsploit to see if there were any public exploits for Magento. Based on how the site appears to work, it is safe to assume it is an eCommerce site.&#x20;

I verified that by checking good ol Google

<figure><img src="../../../.gitbook/assets/image (588).png" alt=""><figcaption></figcaption></figure>

I noticed a particularly interesting exploit

{% embed url="https://www.exploit-db.com/exploits/37977" %}

Essentially this script run a query that takes a username and password and makes that user an admin. It sends the request, while encoded in base64 to a vulnerable URL that can be bypassed????? I need to read the original blog

I modified it as shown in the image below.

<figure><img src="../../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

Now that we are logged in as an admin user, the next step would typically be to see if we can find a way to upload files or edit files in a way that can result in us getting a shell on the box.

I finally was able to find a version of Magento which is 1.9.0.0

<figure><img src="../../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../../.gitbook/assets/image (465).png" alt=""><figcaption></figcaption></figure>

## Foothold

{% embed url="https://blog.scrt.ch/2019/01/24/magento-rce-local-file-read-with-low-privilege-admin-rights/" %}

## Priv Esc

{% embed url="https://gtfobins.github.io/gtfobins/vi/#file-write" %}
