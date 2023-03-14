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

If you don't have a list of valid users, we can use Kerbrute in conjunction with the `jsmith.txt` or `jsmith2.txt` user lists from [Insidetrust](https://github.com/insidetrust/statistically-likely-usernames). This repository contains many different user lists that can be extremely useful when attempting to enumerate users when starting from an unauthenticated perspective.&#x20;
