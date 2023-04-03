# FTP

```bash
ftp> status
# detailed output
ftp> debug
ftp> trace
# recursive listing
ftp> ls -R
# show contents of a file
ftp> get file.txt -
# download a file
ftp> get someFile.txt
# download all available files
wget -m --no-passive ftp://anonymous:anonymous@IP
# upload a file
ftp> put file.txt

# NMAP
sudo nmap -sV -p21 -sC -A 10.129.14.136

# service interaction
nc -nv 10.129.14.136 21
telnet 10.129.14.136 21

# ftp server with TLS/SSL encryption
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

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
