# Driver

First I use Rustscan to get a quick overview of what ports are open

![](<../../../../.gitbook/assets/image (27).png>)

Afterwards I'll use Nmap to gather more information

```bash
nmap -Pn -n -T4 -v -p- -A -sV 10.129.154.147
```

![](<../../../../.gitbook/assets/image (22).png>)

HTTP Port 80,  NetBIOS Port 139, SMB Port 445 and Windows Remote Management over HTTP Port 5985

## Web Fuzzing

Since I see that port 80 is open, I'll run some gobuster scans in the background

```bash
gobuster dir -u driver.htb -w /usr/share/wordlists/dirb/common.txt -q &\
gobuster dir -u driver.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q &\
gobuster dir -u driver.htb -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -q &\
gobuster vhost -u driver.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt -q &
```

![](<../../../../.gitbook/assets/image (54).png>)

There was nothing useful here found so I moved on to see what the website looks like

When going there, we are presented with a sign-in prompt

![](<../../../../.gitbook/assets/image (24).png>)

I try some default credentials such as <mark style="color:green;">`admin:admin`</mark> and we successfully log in

![](<../../../../.gitbook/assets/image (41).png>)

Navigating through the site, we can only access "Firmware Updates" which brings us to this page where we can upload a file

![](<../../../../.gitbook/assets/image (20).png>)
