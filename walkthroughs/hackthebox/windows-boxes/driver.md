# Driver

First I use Rustscan to get a quick overview of what ports are open

![](<../../../.gitbook/assets/image (510).png>)

Afterwards I'll use Nmap to gather more information

```bash
nmap -Pn -n -T4 -v -p- -A -sV 10.129.154.147
```

![](<../../../.gitbook/assets/image (372).png>)

HTTP Port 80,  NetBIOS Port 139, SMB Port 445 and Windows Remote Management over HTTP Port 5985

## Web Fuzzing

Since I see that port 80 is open, I'll run some gobuster scans in the background

```bash
gobuster dir -u driver.htb -w /usr/share/wordlists/dirb/common.txt -q &\
gobuster dir -u driver.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q &\
gobuster dir -u driver.htb -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -q &\
gobuster vhost -u driver.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt -q &
```

![](<../../../.gitbook/assets/image (256).png>)

There was nothing useful here found so I moved on to see what the website looks like

When going there, we are presented with a sign-in prompt

![](<../../../.gitbook/assets/image (587).png>)

I try some default credentials such as <mark style="color:green;">`admin:admin`</mark> and we successfully log in

![](<../../../.gitbook/assets/image (506).png>)

Navigating through the site, we can only access "Firmware Updates" which brings us to this page where we can upload a file

![](<../../../.gitbook/assets/image (563).png>)

{% embed url="https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks" %}

![](<../../../.gitbook/assets/image (556).png>)

![](<../../../.gitbook/assets/image (174).png>)

![](<../../../.gitbook/assets/image (99).png>)

![](<../../../.gitbook/assets/image (611).png>)

![](<../../../.gitbook/assets/image (332).png>)

![](<../../../.gitbook/assets/image (110).png>)

{% embed url="https://book.hacktricks.xyz/pentesting/5985-5986-pentesting-winrm" %}

![](<../../../.gitbook/assets/image (309).png>)

![](<../../../.gitbook/assets/image (276).png>)

{% embed url="https://0xdf.gitlab.io/2021/07/08/playing-with-printnightmare.html" %}

![](<../../../.gitbook/assets/image (278).png>)

![](<../../../.gitbook/assets/image (288).png>)

![](<../../../.gitbook/assets/image (308).png>)

![](<../../../.gitbook/assets/image (98).png>)

![](<../../../.gitbook/assets/image (570).png>)

![](<../../../.gitbook/assets/image (513).png>)

![](<../../../.gitbook/assets/image (539).png>)

![](<../../../.gitbook/assets/image (314).png>)
