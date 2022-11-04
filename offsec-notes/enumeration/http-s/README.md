# HTTP(S)

## Basic Info

```bash
nc -v domain.com 80 # GET / HTTP/1.0
openssl s_client -connect domain.com:443 # GET / HTTP/1.0
```

## Server Versions

```bash
whatweb -a 1 <URL> #Stealthy
whatweb -a 3 <URL> #Aggresive
webtech -u <URL>
webanalyze -host https://google.com -crawl 2
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

```
cmsmap
wpscan
```

## Directory & File Brute Force

### Feroxbuster

{% code overflow="wrap" %}
```bash
feroxbuster -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 150 -x php,jsp,html,txt,.bak,sh,py,pl,c,aspx,asp -o feroxbuster.out -C 404,403
```
{% endcode %}

### Gobuster

<pre class="language-bash"><code class="lang-bash">gobuster dir -u domain.htb -t 100 -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u domain.htb -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
gobuster dir -u domain.htb -t 100 -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
<strong>gobuster vhost -u domain.htb -t 100 -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt
</strong>gobuster dns -d inlanefreight.com -t 100 -w /usr/share/SecLists/Discovery/DNS/namelist.txt</code></pre>

### Dirsearch

```bash
dirsearch -u 10.10.62.196 -w /usr/share/wordlists/dirb/common.txt -x 404 -t 100 
```

### [ffuf](https://github.com/ffuf/ffuf)

```
ffuf -w /path/to/wordlist -u https://target/FUZZ
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
