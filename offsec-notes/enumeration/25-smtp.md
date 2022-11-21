# SMTP

#### banner grab

```
nc -nv 10.11.1.217 25
telnet 10.10.10.10 25
```

#### nmap

{% code overflow="wrap" %}
```
sudo nmap --script=smtp-commands,smtp-enum-users,smtp-ntlm-info,smtp-open-relay,smtp-vuln-cve2010-4344,smtp-vuln-cve2011-1720,smtp-vuln-cve2011-1764 -p 25 10.10.10.10 -Pn -v
```
{% endcode %}

#### MX server

```
dig +short mx domain
```

```
rpcinfo -s 10.11.1.72
rpcinfo -p 10.11.1.72
```

