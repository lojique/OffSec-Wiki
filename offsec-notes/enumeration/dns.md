# 53 - DNS

### Banner Grab

```python
dig version.bind CHAOS TXT @<domain.local/10.10.10.10>
```

### Nmap

```python
nmap -p53 10.10.10.10 --script dns-nsid
```

### Any record

> The record **ANY** will ask the DNS server to **return** all the available **entries** that **it is willing to disclose**

```bash
dig any domain.local/IP @IP
```

### Zone transfer

> This procedure is abbreviated `Asynchronous Full Transfer Zone` (`AXFR`).

```bash
dig axfr @<DNS_IP> #Try zone transfer without domain
dig axfr @<DNS_IP> <DOMAIN> #Try zone transfer guessing the domain
fierce --domain <DOMAIN> --dns-servers <DNS_IP> #Will try to perform a zone transfer against every authoritative name server and if this doesn't work, will launch a dictionary attack
```

### More info

```bash
dig ANY @<DNS_IP> <DOMAIN>     #Any information
dig A @<DNS_IP> <DOMAIN>       #Regular DNS request
dig AAAA @<DNS_IP> <DOMAIN>    #IPv6 DNS request
dig TXT @<DNS_IP> <DOMAIN>     #Information
dig MX @<DNS_IP> <DOMAIN>      #Emails related
dig NS @<DNS_IP> <DOMAIN>      #DNS that resolves that name
dig -x 192.168.0.2 @<DNS_IP>   #Reverse lookup
dig -x 2a00:1450:400c:c06::93 @<DNS_IP> #reverse IPv6 lookup

#Use [-p PORT]  or  -6 (to use ivp6 address of dns)
```

### Using nslookup

```bash
nslookup
> SERVER <IP_DNS> #Select dns server
> 127.0.0.1 #Reverse lookup of 127.0.0.1, maybe...
> <IP_MACHINE> #Reverse lookup of a machine, maybe...
```

### Useful metasploit modules

```bash
auxiliary/gather/enum_dns #Perform enumeration actions
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
