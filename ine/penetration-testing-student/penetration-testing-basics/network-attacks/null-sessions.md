# Null Sessions

Null session attacks can be used to enumerate a lot of information

Attackers can steal information about:

* Passwords
* System users
* System groups
* Running system processes

They are remotely exploitable which means attackers can use their computers to attack a vulnerable Windows machine

This can attack can be used to call remote APIs and remote procedure calls

{% hint style="info" %}
Nowadays Windows is configured to be immune from this kind of attack. However, legacy hosts can still be vulnerable
{% endhint %}

A null session attack exploits an authentication vulnerability for Windows Administrative Shares which lets an attacker connect to a local or remote share without authentication

## Enumerating Windows Shares

### NbtStat

A Windows command line tool that can display information about a target

`nbtstat -A <IP>` displays information about a target

![](<../../../../.gitbook/assets/image (29) (1) (1) (1) (1) (1) (1) (1).png>)

* Here we see the names of the machine running at 10.130.40.80
* The record type <00> tells us that ELS-WINXP is a workstation
* The type "UNIQUE" tells us that the computer must have only one IP address assigned
* The second line contains the workgroup or the domain the computer is joined to
* The third line contains the record type <20> which tells us that the file sharing service is up and running and we can try to get some more information about it

## NET VIEW

Once an attacker knows that a machine has the File Server service running, they can enumerate the shares with NET VIEW

```
NET VIEW <target IP>
```

![](<../../../../.gitbook/assets/image (27) (1) (1) (1) (1) (1).png>)

## Nmblookup

We can also perform shares enumeration from a Linux machine

To perform the same operations of nbstat, we can use nmblookup with the same command

```
nmblookup -A <target IP>
```

![](<../../../../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1).png>)

## Smbclient

An FTP-like client to access Windows shares

```
smbclient -L //10.130.40.80 -N
```

`-N` forces the tool to not ask for a password

![](<../../../../.gitbook/assets/image (5) (1) (1).png>)

Smbclient can detects the very same shares detected by NET VIEW but also displays adminsitrative tools that are hidden when using Windows standard tools (IPC$, Admin$, and C$)

## Checking for Null Sessions

In this example, we'll exploit the IPC$ administrative share by trying to connect to it without valid credentials

### with Windows

This tells Windows to connect to the IPC$ share by using both an empty password and username

```
NET USE \\<target IP>\IPC$ '' \u:''
```

![](<../../../../.gitbook/assets/image (9) (1) (1) (1).png>)

This command does not work with C$

![](<../../../../.gitbook/assets/image (31) (1) (1) (1).png>)

### with Linux

We can do the same checks by using smbclient

```
smbclient \\<target IP>/IPC$ -N
```

![](<../../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1).png>)

## Exploiting Null Sessions

We can exploit null sessions using the WIndows NET command but there are some automated tools we can also leverage

### Enum

[Enum](http://packetstormsecurity.com/search/?q=win32+enum\&s=files) is a command line utility that can retrieve information from a system vulnerable to null session attacks

```
enum -S <target IP> 
```

![](<../../../../.gitbook/assets/image (21) (1) (1) (1) (1) (1).png>)

\-S enumerate shares on a machine

\-U enumerates users

![](<../../../../.gitbook/assets/image (23) (1) (1) (1) (1).png>)

\-P if you need to mount a network authentication attack, you can check the password policy

![](<../../../../.gitbook/assets/image (25) (1) (1) (1) (1) (1).png>)

Checking password policies before running an authentication attack lets you find-tune an attack tool to:

* Prevent accounts locking
* Prevent false positives
* Choose your dictionary or your bruteforcer configuration (know the min/max password length helps you save time)

### Winfo

Another command line [utility](http://packetstormsecurity.com/search/?q=winfo\&s=files) used to automate null session exploitation

the -n switch tells the tool to use null sessions

```
winfo <target IP> -n
```

### Enum4Linux

enum4linux is a PERL script that can perform the same operations of enum and Winfo

By default it performs:

* User enumeration
* Share enumeration
* Group and member enumeration
* Password policy extraction
* OS information detection
* A nmblookup run
* Printer information extraction

## About Null Sessions

Even if by default they are not enabled on modern Microsoft operations systems, you can sometimes find them on enterprise networks because of retro compatibility with legacy systems and applications
