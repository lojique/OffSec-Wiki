# Attacking AD

## PowerView

{% embed url="https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon" %}

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

```
powershell.exe -nop -exec bypass
Import-Module .\PowerView.ps1
Get-NetLoggedon -ComputerName "name"
Get-NetSession -ComputerName "DC"
```

[https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)&#x20;

## Enumeration

```
net user
net user /domain # enum all users in entire domain
net group /domain
```
