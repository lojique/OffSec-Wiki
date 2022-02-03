---
description: A ctf for beginners, can you root me?
---

# RootMe

The room can be accessed [here](https://tryhackme.com/room/easyctf)

\*\*\*Still editing this writeup\*\*\*

## Reconnaissance

### Nmap Scan

```bash
└─$ nmap -Pn -n -sV -A -T4 -v -p22,80 10.10.62.196                      
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: HackIT - Home
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Dirsearch

```bash
dirsearch -u 10.10.62.196 -w /usr/share/wordlists/dirb/common.txt -x 404 -t 100 
```

![](<../../.gitbook/assets/image (15).png>)

Here we can see that we can upload files. So we should be interested in uploading a reverse shell

![](<../../.gitbook/assets/image (1).png>)

And we should be able to execute it here

![](<../../.gitbook/assets/image (53).png>)

Here's the PHP reverse shell. Now I start an netcat listener

![](<../../.gitbook/assets/image (64).png>)

And attempt to load the file, however, php is not permitted

![](<../../.gitbook/assets/image (17).png>)

What we can do, is bypass the filter by renaming the file to something like shell.php.jpg&#x20;

The php reverse shell can be found here:

{% embed url="https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php" %}

![](<../../.gitbook/assets/image (51).png>)

As we can see, it accepted the file!

![](<../../.gitbook/assets/image (34).png>)

## Getting a Shell

We'll start a NetCat listener, go to the uploads directory and click on our file to get that reverse shell!

![](<../../.gitbook/assets/image (62).png>)

Unfortunately that did not work. Maybe we'll try any of these combinations: **php** phtml, .php, .php3, .php4, .php5

This was quite helpful

{% embed url="https://sushant747.gitbooks.io/total-oscp-guide/content/bypass_image_upload.html" %}

Once again, our file was accepted and uploaded

![](<../../.gitbook/assets/image (54).png>)

And there's our shell!

![](<../../.gitbook/assets/image (55).png>)

Here is where the user.txt file is

![](<../../.gitbook/assets/image (29).png>)

## Privilege Escalation

Now for the fun part

First I really wanted to upgrade my shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
(inside the nc session) CTRL+Z;stty raw -echo; fg; ls; export SHELL=/bin/bash; export TERM=screen; stty rows 38 columns 116; reset;
```

Running sudo -l won't help us because it asks for www-data password, which we don't know

Looking at the question, it asks: "Search for files with SUID permission, which file is weird?"

This article helped me better understand SUID:

{% embed url="https://www.hackingarticles.in/linux-privilege-escalation-using-suid-binaries" %}

### **What is SUID Permission?**

**SUID:** Set User ID is a type of permission that allows users to execute a file with the permissions of a specified user. Those files which have suid permissions run with higher privileges.  Assume we are accessing the target system as a non-root user and we found suid bit enabled binaries, then those file/program/command can run with root privileges. ****&#x20;

**How to set suid?**

Basically, you can change the permission of any file either using the “Numerical” method or “Symbolic” method. As result, it will **replace x from s** as shown in the below image which denotes especial execution permission with the higher privilege to a particular file/command. Since we are enabling SUID for Owner (user) therefore **bit 4** or **symbol s** will be added before read/write/execution operation.

![](<../../.gitbook/assets/image (38).png>)

If you execute **ls -al** with the file name and then you observe the small ‘s’ symbol as in the above image, then its means SUID bit is enabled for that file and can be executed with root privileges.

### **How does SUID help in privilege escalation?**

In Linux, some of the existing binaries and commands can be used by non-root users to escalate root access privileges if the SUID bit is enabled. There are some famous Linux / Unix executable commands that can allow privilege escalation: Bash, Cat, cp, echo, find, Less, More, Nano, Nmap, Vim, etc

```bash
find / -perm -u=s -type f 2>/dev/null
```

* **/** denotes  start from the top (root) of the file system and find every directory
* **-perm** denotes search for the permissions that follow
* **-u=s** denotes look for files that are owned by the root user
* **-type** states the type of file we are looking for
* **f** denotes a regular file, not the directories or special files
* **2** denotes to the second file descriptor of the process, i.e. stderr (standard error)
* **>** means redirection
* **/dev/null** is a special filesystem object that throws away everything written into it.

What stands out is /usr/bin/python

![](<../../.gitbook/assets/image (70).png>)

![](<../../.gitbook/assets/image (23).png>)

As we can see, python can be executed as root!

Here's how we can do this

[https://gtfobins.github.io/gtfobins/python/#suid](https://gtfobins.github.io/gtfobins/python/#suid)

![](<../../.gitbook/assets/image (46).png>)

It works!

![](<../../.gitbook/assets/image (47).png>)

Now let's get the flag

![](<../../.gitbook/assets/image (36).png>)

We're done!
