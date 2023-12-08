# ARP Poisoning

ARP Poisoning is a powerful attack you can use to intercept traffic on a switched network\
To send an IP packet, a host needs to know the MAC address of the next hop\
The next hop could be a router, switch or the destination host

To identify the MAC address of a host, computers use the Address Resolution Protocol

![](<../../../../.gitbook/assets/image (46).png>)

After the MAC address resolution is comnplete, hosts save the destination address in their ARP cache table

![](<../../../../.gitbook/assets/image (215).png>)

If an attacker finds a way to manipulate the ARP cache, they will also be able to receive traffic destined to other IP addresses

This can happen because, as long as a destination MAC address is in the ARP cache table, the sender does not need to run ARP to reach that destination host

If an attacker manipulates the ARP tables of the two parties involved in a communication, it will be able to sniff the whole communication, thus peforming a man-in-the-middle (MITM) attack.

This can be done by sending gratuitous ARP replies

During an ARP poisoning attack, three actors are involved:

* Two network nodes (clients, servers, routers, printers,...)
* The attacker

![](<../../../../.gitbook/assets/image (614).png>)

## Gratuitous ARP Replies

The attacker can manipulate other hosts' ARP cache tables by sending gratuitous ARP replies which are unsolicited ARP reply messages

In other words, the attacker sends a reply without waiting for a host to perform a request

The attacker exploits gratuitous ARP messages to tell the victims that they can reach a specific IP address at the attacker's machine MAC address

![](<../../../../.gitbook/assets/image (310).png>)

This must be done on every victim

![](<../../../../.gitbook/assets/image (363).png>)

As soon as the ARP cache table contains fake information, every packet of every communication between the poisoned nodes will be sent to the attacker's machine

The attacker can prevent the poisoned entry from expiring by sending gratuitous ARP replies every 30 seconds or so

## Forwarding and Mangling Packets

As soon as the attacker's machine receives the packets, it must forward them to the correct destination, otherwise the communication between the victim hosts will not work

This allows the hacker to sniff traffic between the poisoned hosts even if the machines sit on a switched network

This activity can go further because the attacker can also change the content of the packets thus manipulating the information exchanged by the two parties

## Local to Remote Man in the Middle

This kind of attack can be used on an entire network and against a router, letting an attacker intercept the communication between a LAN and the Internet

## Dsniff Arpspoof

Dsniff is a collection of tools for network auditing and penetration testing

It includes arpspoof, a utility designed to intercept traffic on a switched LAN

Before running the tool, you have to enable the Linux Kernal IP Forwarding, a feature that transforms a Linux box into a router

By enabling IP forwarding, you tell your machine to forward the packets you intercept to the real destination host

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Then you can run arpspoof

```
arpspoof -i <interface> -t <target> -r <host>
```

Interface is the NIC you want to use (eth0, tap0)

Target and Host are the victims IP addresses

### Example

```
echo 1 > /proc/sys/net/ipv4/ip_forward
arpspoof -i eth0 -t 192.168.4.11 -r 192.168.4.16
```

You can then run Wireshark and intercept the traffic
