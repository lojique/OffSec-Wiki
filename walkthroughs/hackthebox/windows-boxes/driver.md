# Driver

First I use Rustscan to get a quick overview of what ports are open

![](<../../../.gitbook/assets/image (27) (1).png>)

Afterwards I'll use Nmap to gather more information

```bash
nmap -Pn -n -T4 -v -p- -A -sV 10.129.154.147
```

![](<../../../.gitbook/assets/image (22) (1).png>)

HTTP Port 80,  NetBIOS Port 139, SMB Port 445 and Windows Remote Management over HTTP Port 5985

## Web Fuzzing

Since I see that port 80 is open, I'll run some gobuster scans in the background

```bash
gobuster dir -u driver.htb -w /usr/share/wordlists/dirb/common.txt -q &\
gobuster dir -u driver.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q &\
gobuster dir -u driver.htb -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -q &\
gobuster vhost -u driver.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt -q &
```

![](<../../../.gitbook/assets/image (54) (2).png>)

There was nothing useful here found so I moved on to see what the website looks like

When going there, we are presented with a sign-in prompt

![](<../../../.gitbook/assets/image (24) (1).png>)

I try some default credentials such as <mark style="color:green;">`admin:admin`</mark> and we successfully log in

![](<../../../.gitbook/assets/image (41) (1).png>)

Navigating through the site, we can only access "Firmware Updates" which brings us to this page where we can upload a file

![](<../../../.gitbook/assets/image (20) (1).png>)

{% embed url="https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks" %}

![](<../../../.gitbook/assets/image (20) (3).png>)

![](<../../../.gitbook/assets/image (21) (3).png>)

![](<../../../.gitbook/assets/image (29).png>)

![](<../../../.gitbook/assets/image (72) (1).png>)

![](<../../../.gitbook/assets/image (55).png>)

![](<../../../.gitbook/assets/image (64).png>)

{% embed url="https://book.hacktricks.xyz/pentesting/5985-5986-pentesting-winrm" %}

![](<../../../.gitbook/assets/image (7) (1) (1).png>)

![](<../../../.gitbook/assets/image (33) (2) (1).png>)

{% embed url="https://0xdf.gitlab.io/2021/07/08/playing-with-printnightmare.html" %}

![](<../../../.gitbook/assets/image (22).png>)

![](<../../../.gitbook/assets/image (37).png>)

![](<../../../.gitbook/assets/image (15) (2).png>)

![](<../../../.gitbook/assets/image (17) (2).png>)

![](<../../../.gitbook/assets/image (18) (1) (2).png>)

![](<../../../.gitbook/assets/image (38).png>)

![](<../../../.gitbook/assets/image (19) (3).png>)

![](<../../../.gitbook/assets/image (71).png>)
