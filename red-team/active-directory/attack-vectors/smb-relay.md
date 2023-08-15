# SMB Relay

## What is it

A SMB relay attack is where an attacker captures a users NTLM hash and relays its to another machine on the network. Masquerading as the user and authenticating against SMB to gain shell or file access.

## Prerequisites

* SMB Signing disabled on target
* Must be on the local network
* User credentials must have remote login access for example; local admin to the target machine or member of the Domain Administrators group.

## SMB Signing

### What it does

SMB signing verifies the origin and authenticity of SMB packets. Effectively this stops MITM SMB relay attacks from happening. If this is enabled and required on a machine we will not be able to perform a SMB relay attack.

### Scanning for Signing disabled machines

Nmap can be used to check for potential SMB relay targets.&#x20;

```
sudo nmap --script=smb2-security-mode.nse -p 445 192.168.183.0/24 -Pn --open
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

As we can see, the two Windows 10 hosts have 'Message signing enabled but not required' meaning we can perform a SMB relay attack as signing is not required.

## Lab Scenario

In the lab scenario we have two machines

* WS01 (192.168.183.137)(Windows 10)
* WS02 (192.168.183.138)(Windows 10)

We will be capturing a hash on WS01 using LLMNR Poisoning and performing a SMB relay attack to gain shell on WS02.

As per the prerequisites the user account hash we will be capturing (MARVEL\fcastle) is a member of the local administrators group on the machine we will be relaying to WS02.

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## How to perform the attack

### Dumping local SAM Database

Ideally we will use `Responder` for this attack which comes preinstalled on Kali Linux. Before we start `Responder` we need to make a small change to the configuration file.

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

We need to turn off SMB and HTTP servers as we do not want to respond to these protocols as we will be capturing the hash and relaying it to a different tool called `ntlmrelayx.py` from Impacket.

Start Responder with the following command:

```
sudo responder -I eth0 -dwv
```

And then call ntlmrelayx.py from Impacket.

```
sudo impacket-ntlmrelayx -t 192.168.183.138 -smb2support
or
sudo impacket-ntlmrelayx -tf targets.txt -smb2support
```

with both tools listening we can trigger an event using LLMNR.

Responder has caught the user 'fcastle' attempting to browse to a host that does not exists on the network. After DNS has failed to resolve the machine falls back to LLMNR which in this case we have caught the hash and relayed it over to ntlmrelayx.py.

ntlmrelayx.py then forwards the hash over to the machines specified with the `-t` switch which in this case is 192.168.183.138 or WS02.

As the user 'fcastle' is an administrator on WS02, `ntlmrelayx.py` has allowed us to dump the hashes in the SAM database.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

We can then take these hashes and crack them or we can even attempt a pass-the-hash attack and attempt to gain a shell with the NTLMv2 hash on a different machine on the network.

### Interactive Shell

We can gain a shell with the same manner above except this time we specify the `-i` switch in ntlmrelayx.py.

```
sudo impacket-ntlmrelayx -t 192.168.183.138 -smb2support -i
or
sudo impacket-ntlmrelayx -tf targets.txt -smb2support -i
```

When we get a successful authentication message in ntlmrelayx.py we will need to open a netcat bind shell on the localhost and port specified in the ntlmrelayx.py output.

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Start netcat with a localhost address and the port specified in the output above.

```
nc 127.0.0.1 <port>
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## Resource

{% embed url="https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/adversary-in-the-middle/smb-relay" %}
