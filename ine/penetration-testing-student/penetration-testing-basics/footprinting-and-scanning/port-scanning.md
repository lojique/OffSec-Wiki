# Port Scanning

## Nmap

* TCP Connect scans get recorded in daemon logs since from an app POV, the scan probes look like a legit connection
* SYN scans are stealthy because the scanner does not perform a full handshake, but only sends a SYN packet and analyzes the response.
  * RST = closed; ACK = open

### TCP

* **Open** port: _SYN --> SYN/ACK --> RST_
* **Closed** port: _SYN --> RST/ACK_
* **Filtered** port: _SYN --> \[NO RESPONSE]_
* **Filtered** port: _SYN --> ICMP message_

```bash
## Nmap fast scan for the most 1000tcp ports used
nmap -sV -sC -O -T4 -n -Pn -oA fastscan <IP> 
## Nmap fast scan for all the ports
nmap -sV -sC -O -T4 -n -Pn -p- -oA fullfastscan <IP> 
## Nmap fast scan for all the ports slower to avoid failures due to -T4
nmap -sV -sC -O -p- -n -Pn -oA fullscan <IP>
```

### Bypassing Firewalls/Blocked Pings

```
nmap -Pn
```

### Common Ports w/ Service

![](<../../../../.gitbook/assets/image (10) (1).png>)

### Spotting a Firewall

![version not recognized regardless of open http port](<../../../../.gitbook/assets/image (6).png>)

![service type not recognized. "tcpwrapped" means TCP handshake completed, but remote host closed the connection w/o receiving any data](<../../../../.gitbook/assets/image (7).png>)

You can use `--reason` to show an explanation of why a port is marked open or closed

## Masscan

Designed to deal with large network and to scan thousands of IP addresses at once

Faster than Nmap but a bit less accurate
