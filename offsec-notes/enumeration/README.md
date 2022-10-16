# Enumeration

## Passive Information Gathering

* netcraft
* recon-ng

## Nmap

```bash
# stealth/SYN scan
sudo nmap -sS 10.10.10.10
# TCP connect scan
nmap -sT 10.10.10.10
# UDP scan
sudo nmap -sU 10.11.1.115
sudo nmap -sS -sU 10.11.1.115
# host discovery/ping scan/skip port scan
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
# skip DNS resolution (for simple scans against a large number of hosts)
sudo nmap -n 10.10.10.10
# skip ping scan
sudo nmap -Pn 10.10.10.10
# track how sent packets are handled on filtered ports
sudo nmap -p 445 --packet-trace -n --disable-arp-ping -Pn 10.10.10.10
# OS fingerprinting
sudo nmap -O 10.11.1.220
# banner grabbing/service enumeration
nmap -sV -sT -A 10.11.1.220
# vuln scan
sudo nmap --script vuln 10.11.1.10 
```

## Web

### Gobuster

```bash
gobuster dir -u driver.htb -w /usr/share/wordlists/dirb/common.txt -U admin -P admin -q &\
gobuster dir -u driver.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -U admin -P admin -q &\
gobuster dir -u driver.htb -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -U admin -P admin -q &\
gobuster vhost -u driver.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt -U admin -P admin -q &\
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt &
```

### Dirsearch

```bash
dirsearch -u 10.10.62.196 -w /usr/share/wordlists/dirb/common.txt -x 404 -t 100 
```

### [ffuf](https://github.com/ffuf/ffuf)

```
ffuf -w /path/to/wordlist -u https://target/FUZZ
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
