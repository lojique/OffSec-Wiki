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

![](<../../../../.gitbook/assets/image (29).png>)

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

![](<../../../../.gitbook/assets/image (27).png>)

## Nmblookup

We can also perform shares enumeration from a Linux machine

To perform the same operations of nbstat, we can use nmblookup with the same command

```
nmblookup -A <target IP>
```

![](<../../../../.gitbook/assets/image (25).png>)

## Smbclient

