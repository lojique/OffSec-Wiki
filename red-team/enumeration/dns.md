# DNS

DNS holds interesting information for an organization. We can understand how a company operates and the services they provide, as well as third-party service providers like emails.

```bash
nmap -p53 -Pn -sV -sC 10.10.110.213

# SOA record
dig soa www.inlanefreight.com

# NS query
dig ns inlanefreight.htb @10.129.14.128

# Version query
dig CH TXT version.bind 10.129.120.85

# Any query
dig any inlanefreight.htb @10.129.14.128

# Zone transfer
dig axfr inlanefreight.htb @10.129.14.128
dig AXFR @ns1.inlanefreight.htb inlanefreight.htb
fierce --domain zonetransfer.me
fierce --domain <DOMAIN> --dns-servers <DNS_IP> #Will try to perform a zone transfer against every authoritative name server and if this doesn't work, will launch a dictionary attack

# Reverse lookup
dig -x 192.168.0.2 @<DNS_IP>
# Reverse IPv6 lookup
dig -x 2a00:1450:400c:c06::93 @<DNS_IP> 

# Interal zone transfer
dig axfr internal.inlanefreight.htb @10.129.14.128

# subdomain bruteforcing
./subfinder -d inlanefreight.com -v #https://github.com/projectdiscovery/subfinder
for sub in $(cat /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb


msfconsole -q; use auxiliary/gather/enum_dns #Perform enumeration actions
```

An excellent alternative is a tool called [Subbrute](https://github.com/TheRook/subbrute). This tool allows us to use self-defined resolvers and perform pure DNS brute-forcing attacks during internal penetration tests on hosts that do not have Internet access.

**Subbrute**

```shell
lojique@htb[/htb]$ git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
lojique@htb[/htb]$ cd subbrute
lojique@htb[/htb]$ echo "ns1.inlanefreight.com" > ./resolvers.txt
lojique@htb[/htb]$ ./subbrute inlanefreight.com -s ./names.txt -r ./resolvers.txt

Warning: Fewer than 16 resolvers per process, consider adding more nameservers to resolvers.txt.
inlanefreight.com
ns2.inlanefreight.com
www.inlanefreight.com
ms1.inlanefreight.com
support.inlanefreight.com

<SNIP>
```

Different `DNS records` are used for the DNS queries, which all have various tasks. Moreover, separate entries exist for different functions since we can set up mail servers and other servers for a domain.

| **DNS Record** | **Description**                                                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `A`            | Returns an IPv4 address of the requested domain as a result.                                                                                                                                                                                      |
| `AAAA`         | Returns an IPv6 address of the requested domain.                                                                                                                                                                                                  |
| `MX`           | Returns the responsible mail servers as a result.                                                                                                                                                                                                 |
| `NS`           | Returns the DNS servers (nameservers) of the domain.                                                                                                                                                                                              |
| `TXT`          | This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam. |
| `CNAME`        | This record serves as an alias. If the domain www.hackthebox.eu should point to the same IP, and we create an A record for one and a CNAME record for the other.                                                                                  |
| `PTR`          | The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.                                                                                                                                     |
| `SOA`          | Provides information about the corresponding DNS zone and email address of the administrative contact.                                                                                                                                            |

###

### Banner Grab

```python
dig version.bind CHAOS TXT @<domain.local/10.10.10.10>
```

### Nmap

```python
nmap -p53 10.10.10.10 --script dns-nsid
```

### Using nslookup

```bash
nslookup
> SERVER <IP_DNS> #Select dns server
> 127.0.0.1 #Reverse lookup of 127.0.0.1, maybe...
> <IP_MACHINE> #Reverse lookup of a machine, maybe...
```

### Useful nmap scripts

```bash
#Perform enumeration actions
nmap -n --script "(default and *dns*) or fcrdns or dns-srv-enum or dns-random-txid or dns-random-srcport" <IP>
```

### DNS - Reverse BF

```bash
dnsrecon -r 127.0.0.0/24 -n <IP_DNS>  #DNS reverse of all of the addresses
dnsrecon -r 127.0.1.0/24 -n <IP_DNS>  #DNS reverse of all of the addresses
dnsrecon -r <IP_DNS>/24 -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d active.htb -a -n <IP_DNS> #Zone transfer
```

