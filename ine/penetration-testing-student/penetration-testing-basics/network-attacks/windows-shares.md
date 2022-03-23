# Windows Shares

## NetBIOS

Network Basic Input Output System

Servers and clients use NetBIOS when viewing network shares on the local area network

![](<../../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png>)

NetBIOS can supply some of the following information when querying a computer:

* Hostname
* NetBIOS name
* Domain
* Network shares

![Structure of NetBIOS](<../../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1).png>)

The NetBIOS layer sits between the application layer and the IP layer

UDP is used to perform NetBIOS name resolution and to carry other one-to-many datagram-based communications

* By using these datagrams, a host can send small messages to many hosts

Heavy traffic (ex: a file copy) relies on TCP by using NetBIOS sessions

{% hint style="warning" %}
Different features use different ports and transport protocols
{% endhint %}

When a Windows machine browses a network, it uses NetBIOS:

* **Datagrams** to list the shares and the machines
* **Names** to find workgroups
* **Sessions** to transmit data to and from a Windows share

## Shares

A Windows machine can share a file or a directory on the network

This allows local and remote users to access and possibly modify it

* **Example**: A file server in an office lets users open and edit the documents of their own department, while it lets everyone read but not modify public information files

Obviously this feature is very useful in a network environment and can either be extremely useful, if used properly, or extremely dangerous when configured improperly

It's quite simple to create network shares. Users just need to turn on the _File and Printer Sharing_ service and then they can start choosing directories or files to share

Users can also set permissions on a share, choosing who can perform operations such as reading, writing and modifying permissions

Starting from Win Vista, users can choose to share a single file or use the _Public_ directory. When sharing a single file, they can choose local or remote users to share the file with

When using the _Public_ directory, they can choose which local users can access the files on the share, but they can only allow everyone or no one in the network to access the share

## UNC Paths

An unauthorized user can access shares by using **Universal Naming Convention paths**

The format is as follows:

```
\\ServerName\ShareName\file.nat
```

## Administrative Shares

There are some special default administrative shares which are used by sysadmins and Windows itself:

* **\\\ComputerName\C$** lets an admin access a volume on the local machine. Every volume has a share (C$, D$, F$, etc.)
* **\\\ComputerName\admin$** points to the windows installation directory
* **\\\ComputerName\ipc$** is used for inter-process communication. You cannot browse it via Windows Explorer

## Badly Configured Shares

Accessing a share means having access to the resources of the computer hosting

So, badly configured shares exploitation can lead to:

* Information disclosure
* Unauthorized file access
* Information leakage used to mount a targeted attack
