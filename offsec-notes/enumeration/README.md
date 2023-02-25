# Enumeration

## Passive Information Gathering

* netcraft
* recon-ng

## Host Discovery&#x20;

```
arp-scan -l
netdiscover -r <ATTACKER IP>.0/24
```

### Domain Information

* [https://crt.sh/](https://crt.sh/) | certificate transparency
  *   {% code overflow="wrap" %}
      ```shell-session
      curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
      ```
      {% endcode %}



      <figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>
  *   {% code title="filter by unique subdomains" overflow="wrap" %}
      ```shell-session
      curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
      ```
      {% endcode %}

      <figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>
  *   {% code title="identify company hosted servers" overflow="wrap" %}
      ```shell-session
      for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
      ```
      {% endcode %}



      <figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

#### Shodan

*   <pre class="language-shell-session"><code class="lang-shell-session"><strong>lojique@htb[/htb]$ for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
    </strong>lojique@htb[/htb]$ for i in $(cat ip-addresses.txt);do shodan host $i;done
    </code></pre>



    <figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

#### dig | DNS Records

```shell-session
dig any inlanefreight.com
```

<figure><img src="../../.gitbook/assets/image (20) (4).png" alt=""><figcaption></figcaption></figure>

### Cloud Resources

{% embed url="https://domain.glass/" %}

{% embed url="https://buckets.grayhatwarfare.com/" %}

Google Dorking --> `intext:domainname` `inurl:amazonaws.com` `inurl:blob.core.windows.net`

## Active Information Gathering

### Nmap

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
# Firewall and IDS/IPS Evasion 
sudo nmap -p -Pn -n --disable-arp-ping -D RND:5 10.10.10.10 # Scan by Using Decoys
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0 # Scan by Using Different Source IP
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

####
