# SMB

## Using NMAP&#x20;

### Scan for popular RCE exploits.

```
sudo nmap -p 139,445 --script smb-vuln* <ip-addr> -oA nmap/smb-vuln
```

### Identify the SMB/OS version.

```
nmap -v -p 139,445 --script=smb-os-discovery.nse <ip-addr>
```

### Enumerate users once you have valid credentials:

```
sudo nmap --script=smb-enum-users -p 445 192.168.1.2 --script-args smbuser=<user>,smbpass=<password>
```

## Using NBTSCAN&#x20;

### To scan a subnet for list of hostnames:

```
nbtscan -v <targetRange>
```

## Using SMBMAP&#x20;

### To list out the shares and associated permissions with Anonymous Access:

```
smbmap -H <ip-addr>
```

### To list out the shares recursively:

```
smbmap -R <sharename> -H <ip-addr>
```

### To list shares as an authenticated user:

```
smbmap -d <domain> -u <username> -p <password> -H <ip-addr>
```

### To list the shares as a Guest user, just provide a username that doesn’t exist.

```
smbmap -u DoesNotExist -H <ip-addr>
```

### To download a particular file

```
smbmap -R <sharename> -H <ip-addr> -A <filename> -q
```

## Using SMBCLIENT&#x20;

### To list out the shares:

```
smbclient -L \\\\<ip-addr>
```

### To connect to shares:

```
sudo smbclient \\\\<ip-addr>\\<share>
```

### Downloading files:&#x20;

Once connected, you can download files. You’ll want to disable interactive prompts and turn recursive mode ON.

```
smb: /> prompt
smb: /> recurse
smb: /> mget *
```

## Using RPCCLIENT

### **Testing for Null or Authenticated Sessions:**

To test for null sessions, you can use the following command. If it connects, then you’ll be able to issue rpc client commands for further enumeration.

```
rpcclient -U "" -N [ip]
```

### **Have valid credentials? Use them to connect:**

```
rpcclient -U <user> 10.10.10.193
```

Once connected, there are various queries you can run.

### **To enumerate printers:**

_`enumprinters`_

### **To enumerate users and groups:**

`enumdomusers`

`enumdomgroups`

**The above command will output user/group RIDs. You can pass those into further queries like:**\
`querygroup`` `**`<RID>`**\
`querygroupmem`` `**`<RID>`**\
`queryuser`` `**`<RID>`**

## Using ENUM4LINUX

The following command will attempt to establish a null session with the target and then use RPC to extract useful information.

```
enum4linux -a [ip]
```

Example output is long, but some highlights to look for:&#x20;

* Listing of file shares and printers
* Domain/Workgroup information
* Password policy information
* RID cycling output to enumerate users and groups

## Using Metasploit&#x20;

### Bruteforcing credentials:

```
use auxiliary/scanner/smb/smb_login
set BLANK_PASSWORDS true
set PASS_FILE /usr/share/seclists/Passwords/Common-Credentials/best15.txt
set USER_FILE /usr/share/seclists/Usernames/top-usernames-shortlist.txt
set RHOSTS <ipAddr>
```

## Mounting SMB Shares in Linux&#x20;

The following command will mount the remote fileshare to /mnt/smb/ (this directory must exist first) and prompt you for the password.

```
mount -t cifs -o username=<user> //<targetIP>/<shareName> /mnt/smb/
```

Another way to mount a share from Linux is as follows:

```
sudo mount.cifs //<targetIP>/share /mnt/share username=,password=
```

## Enumeration from Windows Utilities&#x20;

### To get the Name Table:

```
nbtstat -A <targetIP>
```

### To see a list of running shares:

```
net view <targetIP>
```

### You can map a share to a drive letter, such as K:

```
net use K: \\<targetIP>\share
```

### Testing for null session:

```
net use \\<targetIP>\IPC$ "" /u:""
```

## Gaining a Shell&#x20;

Once you have valid credentials on the machine, or a valid NTLM hash, you can leverage the following guide to gain a shell.

{% embed url="https://infinitelogins.com/2020/09/05/popping-remote-shells-pth-winexe-on-windows" %}



## Resource

{% embed url="https://infinitelogins.com/2020/06/17/enumerating-smb-for-pentesting" %}

{% embed url="https://0xdf.gitlab.io/2018/12/02/pwk-notes-smb-enumeration-checklist-update1.html#manual-inspection" %}
