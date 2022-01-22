---
description: >-
  This exercise will help you understand how to look for Null Sessions in a
  legacy environment using various tools and access the files over SMB.
---

# Lab - Null Session

## Lab Environment

In this lab environment, the user is going to get access to a Kali GUI instance. SMB service is running on the target machine available at [demo.ine.local](http://demo.ine.local).

**Objective:** Scan the target machine and find the 3 flags:

1. Flag 1 is present in the file named **flag\_1** in a publicly accessible share.
2. Flag 2 is present in a directory named **flag\_2** in a non-browsable share of one of the users.
3. Flag 3 is present in a file named **flag\_3** in yet another non-browsable share.

## Instructions

* **Use share enumeration wordlist:** /root/Desktop/wordlists/100-common-passwords.txt

![0](https://assets.ine.com/content/pta-labs/14\_null\_session/0.png)

## Tools

The best tools for this lab are: - enum4linux - Nmap - nmblookup \[optional] - smbclient - smbmap - Terminal

## My Solution

### Flag 1

Flag 1 is present in the file named **flag\_1** in a publicly accessible share.
