# Enumeration

## Passive Information Gathering

* netcraft
* recon-ng

## Web

### Gobuster

```bash
gobuster dir -u driver.htb -w /usr/share/wordlists/dirb/common.txt -U admin -P admin -q &\
gobuster dir -u driver.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -U admin -P admin -q &\
gobuster dir -u driver.htb -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -U admin -P admin -q &\
gobuster vhost -u driver.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt -U admin -P admin -q &\
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt &
```

### Banner Grabbing/Web Server Headers

```bash
curl -IL https://www.inlanefreight.com
```

### EyeWitness

Can be used to take screenshots of target web applications, fingerprint them, and identify possible default credentials

```bash
./EyeWitness.py -f filename --timeout optionaltimeout
```

### Whatweb

```
whatweb 10.10.10.10
whatweb --no-errors 10.10.10.0/24
```
