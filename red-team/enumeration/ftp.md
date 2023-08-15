# FTP

| **Setting**               | **Description**                                                                  |
| ------------------------- | -------------------------------------------------------------------------------- |
| `dirmessage_enable=YES`   | Show a message when they first enter a new directory?                            |
| `chown_uploads=YES`       | Change ownership of anonymously uploaded files?                                  |
| `chown_username=username` | User who is given ownership of anonymously uploaded files.                       |
| `local_enable=YES`        | Enable local users to login?                                                     |
| `chroot_local_user=YES`   | Place local users into their home directory?                                     |
| `chroot_list_enable=YES`  | Use a list of local users that will be placed in their home directory?           |
| `hide_ids=YES`            | All user and group information in directory listings will be displayed as "ftp". |
| `ls_recurse_enable=YES`   | Allows the use of recurse listings.                                              |

```bash
ftp 10.129.14.136
ls -R # recursive listing
get filename.txt - # cat file without downloading locally
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136 # download all files
put filename.txt # upload a file

ftp> status
# detailed output
ftp> debug
ftp> trace
```

### Footprinting

```bash
sudo nmap --script-updatedb
find / -type f -name ftp* 2>/dev/null | grep scripts
sudo nmap -sC -sV -p 21 192.168.2.142
nc -nv 10.129.14.136 21
telnet 10.129.14.136 21
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

**FTP Bounce Attack**

An FTP bounce attack is a network attack that uses FTP servers to deliver outbound traffic to another device on the network. The attacker uses a `PORT` command to trick the FTP connection into running commands and getting information from a device other than the intended server.

Consider we are targetting an FTP Server `FTP_DMZ` exposed to the internet. Another device within the same network, `Internal_DMZ`, is not exposed to the internet. We can use the connection to the `FTP_DMZ` server to scan `Internal_DMZ` using the FTP Bounce attack and obtain information about the server's open ports. Then, we can use that information as part of our attack against the infrastructure.

!\[\[ftp\_bounce\_attack.png]] Source: [https://www.geeksforgeeks.org/what-is-ftp-bounce-attack/](https://www.geeksforgeeks.org/what-is-ftp-bounce-attack/)

The `Nmap` -b flag can be used to perform an FTP bounce attack:

```shell
lojique@htb[/htb]$ nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-27 04:55 EDT
Resolved FTP bounce attack proxy to 10.10.110.213 (10.10.110.213).
Attempting connection to ftp://anonymous:password@10.10.110.213:21
Connected:220 (vsFTPd 3.0.3)
Login credentials accepted by FTP server!
Initiating Bounce Scan at 04:55
FTP command misalignment detected ... correcting.
Completed Bounce Scan at 04:55, 0.54s elapsed (1 total ports)
Nmap scan report for 172.17.0.2
Host is up.

PORT   STATE  SERVICE
80/tcp open http

<SNIP>
```
