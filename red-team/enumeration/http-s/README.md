# HTTP(S)

### Basic Info

```bash
nc -v domain.com 80 # GET / HTTP/1.0
openssl s_client -connect domain.com:443 # GET / HTTP/1.0
```

### Nmap

```bash
nmap -vv --reason -Pn -T4 -sV -p 80 --script="banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" 10.10.10.10
```

### Server Versions

```bash
whatweb -a 1 <URL> #Stealthy
whatweb -a 3 <URL> #Aggresive
whatweb --no-errors 10.10.10.0/24
webtech -u <URL>
webanalyze -host https://google.com -crawl 2
```

### Secure Client-Initated Renegotiation

```bash
# Enter the below and press R+Enter many many times, if you can renegotiate >10-12 times, its vulnerable
openssl s_client -connect host:port
```

### Webfuzzing

**Feroxbuster**

```bash
feroxbuster -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 50 -x php,txt,aspx -o feroxbuster.out -C 404,403
```

**Gobuster**

```bash
gobuster dir -u domain.htb -t 100 -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
gobuster vhost -u domain.htb -t 100 -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt
gobuster dns -d inlanefreight.com -t 100 -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

**FFUF**

```bash
ffuf -w /path/to/wordlist -u https://target/FUZZ

# directory fuzing
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ

# extension fuzzing
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ

# page fuzzing
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php

# recursive scanning
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v

# subdomain fuzzing
ffuf -w /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.academy.htb/
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/shubs-subdomains.txt:FUZZ -u http://FUZZ.academy.htb/

# vhost fuzzing
ffuf -w /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'

# GET request fuzzing
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx

# POST request fuzzing
# Tip: In PHP, "POST" data "content-type" can only accept "application/x-www-form-urlencoded". So, we can set that in "ffuf" with "-H 'Content-Type: application/x-www-form-urlencoded'".
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx

curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'

# value fuzzing
for i in $(seq 1 1000); do echo $i >> ids.txt; done

ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx

curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'

# ignore comments in wordlist
-ic

# filter response size
-fs
```

**Alias function**

```bash
    ffuff() {
      seclists='/home/lojique/HTB/SecLists'
    
      if [[ $1 == "short" ]]; then
        command ffuf -u "$2/FUZZ" -w $seclists/Discovery/Web-Content/quickhits.txt $3 $4 $5 $6 $7 $8 $9 
      
      elif [[ $1 == "long" ]]; then
        command ffuf -u "$2/FUZZ" -w $seclists/Discovery/Web-Content/directory-list-2.3-medium.txt $3 $4 $5 $6 $7 $8 $9 
      
      elif [[ $1 == "http_vhost" ]]; then
        command ffuf -u "http://$2/" -H "Host: FUZZ.$2" -w $seclists/Discovery/DNS/subdomains-top1million-110000.txt $3 $4 $5 $6 $7 $8 $9 
      elif [[ $1 == "https_vhost" ]]; then
        command ffuf -u "https://$2/" -H "Host: FUZZ.$2" -w $seclists/Discovery/DNS/subdomains-top1million-110000.txt $3 $4 $5 $6 $7 $8 $9 
    
      elif [[ $1 == "lfi" ]]; then
        command ffuf -u "$2/FUZZ" -w $seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest-huge.txt $3 $4 $5 $6 $7 $8 $9 
      
      else; 
        echo '
          usage:\n
          ffuff short http(s)://whatever.domain\n
          ffuff long http(s)://whatever.domain\n
          ffuff http_vhost whatever.domain\n
          ffuff https_vhost whatever.domain\n
          ffuff lfi http(s)://whatever.domain/lfi/endpoint
        '
     
      fi
    }
```

#### Dirsearch

```bash
dirsearch -u 10.10.62.196 -w /usr/share/wordlists/dirb/common.txt -x 404 -t 100 
```

#### Add DNS Record

```bash
sudo sh -c 'echo "SERVER_IP  academy.htb" >> /etc/hosts'
echo "SERVER_IP  academy.htb" | sudo tee -a /etc/hosts
```

### Automated Scanners

```bash
nikto -h <URL>
whatweb -a 4 <URL>
wapiti -u <URL>
W3af
nuclei -ut && nuclei -target <URL>
```

#### CMS Scanners

```bash
cmsmap
wpscan --url domain.local --enumerate ap,at,cb,dbe -o domain-wpscan
```

### wfuzz

```
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/IIS.fuzz.txt http://10.10.10.103:80/FUZZ
```

### EyeWitness

Can be used to take screenshots of target web applications, fingerprint them, and identify possible default credentials

```bash
./EyeWitness.py -f filename --timeout optionaltimeout
```

## Curl

```
curl -sSik http://10.10.10.10
curl -sSikf http://10.10.10.10
```
