# Subdomain Enumeration

## Google

```
site: domain.com
```

## DNSdumpster

{% embed url="https://dnsdumpster.com" %}
passive subdomain enumeration that utilizes data from google-indexed subdomains, but also checks sites like Bing or VirusTotal for similar info
{% endembed %}

## Sublist3r

{% embed url="https://github.com/aboul3la/Sublist3r" %}

```bash
sublist3r -d example.com
```

## Amass

```
snap run amass -ip -d example.com
```

{% embed url="https://www.kali.org/tools/amass" %}

{% embed url="https://github.com/OWASP/Amass" %}

## DNSRecon

```
dnsrecon -d example.com -D /usr/share/wordlists/dnsmap.txt -t brt
```

{% embed url="https://www.kali.org/tools/dnsrecon" %}

{% embed url="https://github.com/darkoperator/dnsrecon" %}

## Certificate Databases

{% hint style="info" %}
&#x20;These offer a searchable database of certificates that shows current and historical results of subdomains
{% endhint %}

{% embed url="https://crt.sh" %}

{% embed url="https://transparencyreport.google.com/https/certificates" %}
