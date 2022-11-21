---
description: Simple Network Management Protocol (SNMP) on UDP
---

# 161 - SNMP

Commonly used SNMP protocols 1, 2, and 2c offer no traffic encryption, meaning that SNMP information and credentials can be easily intercepted over a local network. Traditional SNMP protocols also have weak authentication schemes and are commonly left configured with default public and private community strings.

```
sudo nmap -sU --open -p 161 10.11.1.1-254 -oG open-snmp.txt
```

We can use a tool such as [_onesixtyone_](http://www.phreedom.org/software/onesixtyone/), which will attempt a brute force attack against a list of IP addresses. First we must build text files containing community strings and the IP addresses we wish to scan

```bash
echo public > community
echo private >> community
echo manager >> community

for ip in $(seq 1 254); do echo 10.11.1.$ip; done > ips

onesixtyone -c community -i ips
```

Once we find SNMP services, we can start querying them for specific MIB data that might be interesting.



### SNMP Management Info Base (MIB)

|                        |                            |
| ---------------------- | -------------------------- |
| 1.3.6.1.2.1.25.1.6.0   | System Processes           |
| 1.3.6.1.2.1.25.4.2.1.2 | Running Programs           |
| 1.3.6.1.2.1.25.4.2.1.4 | Processes Path             |
| 1.3.6.1.2.1.25.2.3.1.4 | Storage Units              |
| 1.3.6.1.2.1.25.6.3.1.2 | Software Name              |
| 1.3.6.1.4.1.77.1.2.25  | User Accounts              |
| 1.3.6.1.2.1.6.13.1.3   | <p>TCP Local Ports<br></p> |

```bash
# enumerate entire MIB Tree
snmpwalk -c public -v1 -t 10 10.11.1.14
# enumerate windows users
snmpwalk -c public -v1 10.11.1.14 1.3.6.1.4.1.77.1.2.25
# enumerate running windows processes
snmpwalk -c public -v1 10.11.1.73 1.3.6.1.2.1.25.4.2.1.2
# enumerate open TCP ports
snmpwalk -c public -v1 10.11.1.14 1.3.6.1.2.1.6.13.1.3
# enumerate installed software
snmpwalk -c public -v1 10.11.1.50 1.3.6.1.2.1.25.6.3.1.2
```
