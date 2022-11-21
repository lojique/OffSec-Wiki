# 21 - FTP

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
