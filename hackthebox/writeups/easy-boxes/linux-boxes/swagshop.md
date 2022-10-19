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

<figure><img src="../../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

### webfuzz

```bash
dirsearch -u http://10.10.10.140 -w /usr/share/dirb/wordlists/common.txt -t 150 -x 404
```

<figure><img src="../../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

Ran dirsearch and checked out the first item which was `http://10.10.10.140/app/`

From there, I discovered a local.xml file:

<figure><img src="../../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Its contents reveal something interesting

<figure><img src="../../../../.gitbook/assets/image (67).png" alt=""><figcaption><p>local.xml file</p></figcaption></figure>

Based on the content shown in the image above, it would seem likely these credentials would work for mysql. However, because mysql wasn't shown in our  nmap scan, perhaps this is running internally.

```
root
fMVWh7bDHpgZkyfqQXreTjU9
```

### searchsploit

<figure><img src="../../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

I wanted to double check if this was an eCommerce site and I believe it is based on the below screenshots. This can help narrow my search results.

<figure><img src="../../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

After some time, this led me nowhere and I was pretty stuck.

So why was I stuck and what could I have done to prevent this? I believe I was so focused on quickly finding an exploit, that I didn't make sure I thoroughly fuzzed the URL

Had I done that, I would've found this much quicker:

<figure><img src="../../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

Revisiting searchsploit, I decided to try out what was available

I ended up using this one here which essentially creates an admin account with the username `forme` and password `forme` if the magento version is vulnerable

{% embed url="https://www.exploit-db.com/exploits/37977" %}

It's worth reading about the vulnerability here:

{% embed url="https://blog.checkpoint.com/2015/04/20/analyzing-magento-vulnerability/" %}

I modified the script a little bit

<figure><img src="../../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

This script send a based64 encoded request of a query that adds a user and password to admin. If it works, then we should be able to log into the Magento Admin Panel

I run the script and am told it worked

<figure><img src="../../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

I verify this

<figure><img src="../../../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

And it worked!

Scrolling down we finally can confirm a version

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

