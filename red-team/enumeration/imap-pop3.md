# IMAP/POP3

```bash
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC

host -t MX hackthebox.eu
dig mx inlanefreight.com | grep "MX" | grep -v ";"
host -t A mail1.inlanefreight.htb.

curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd

openssl s_client -connect 10.129.14.128:pop3s
openssl s_client -connect 10.129.14.128:imaps

hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3
```
