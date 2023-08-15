# RSYNC

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync" %}

{% embed url="https://academy.hackthebox.com/module/112/section/1240" %}

```shell
sudo nmap -sV -p 873 127.0.0.1

# Probing for Accessible Shares
nc -nv 127.0.0.1 873

# Enumerating an open share
rsync -av --list-only rsync://127.0.0.1/dev
```
