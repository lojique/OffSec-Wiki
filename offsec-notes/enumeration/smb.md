# 139/445 - SMB

## Using NMAP&#x20;

```
ls -1 /usr/share/nmap/scripts/smb*
```

```
sudo nmap -p 139,445 -vv -Pn --script=smb-vuln-cve2009-3103.nse,smb-vuln-ms06-025.nse,smb-vuln-ms07-029.nse,smb-vuln-ms08-067.nse,smb-vuln-ms10-054.nse,smb-vuln-ms10-061.nse,smb-vuln-ms17-010.nse
```

```
sudo nmap -p 139,445 --script smb-vuln* <ip-addr> -oA nmap/smb-vuln
```

```
nmap -v -p 139,445 --script=smb-os-discovery.nse <ip-addr>
```

```
sudo nmap --script=smb-enum-users -p 445 192.168.1.2 --script-args smbuser=<user>,smbpass=<password>
```

## SMBMAP&#x20;

```
# anon access
smbmap -H <ip-addr>
# list shares recursively
smbmap -R <sharename> -H <ip-addr>
smbmap -d <domain> -u <username> -p <password> -H <ip-addr>
smbmap -u DoesNotExist -H <ip-addr>
# download a file
smbmap -R <sharename> -H <ip-addr> -A <filename> -q
```

## SMBClient

```
smbclient -L \\\\<ip-addr>
smbclient -L \\<IP> --user=domain/user%password
smbclient -L \\<IP> -U user
smbclient \\\\<ip-addr>\\<share>
smbclient //$IP/profiles$ -c ls | awk '{print $1}' > users.lst
```

### Downloading files:&#x20;

Once connected, you can download files. You’ll want to disable interactive prompts and turn recursive mode ON.

```
smb: /> prompt
smb: /> recurse
smb: /> mget *
```

## Using ENUM4LINUX-NG

The following command will attempt to establish a null session with the target and then use RPC to extract useful information.

```
enum4linux-ng -A [ip]
```

## Changing a User's Password

```
smbpasswd -U user -r domain.local
impacket-smbpasswd user:oldPassword@domain.local -newpass P@ssw0rd123
```
