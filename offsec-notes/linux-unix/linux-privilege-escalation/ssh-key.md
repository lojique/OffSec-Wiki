# SSH Key

## Sensitive files

```bash
find / -name authorized_keys 2> /dev/null
find / -name id_rsa 2> /dev/null
...
```



If we have read access over the `.ssh` directory for a specific user, we may read their private ssh keys found in `/home/user/.ssh/id_rsa` or `/root/.ssh/id_rsa`, and use it to log in to the server. If we can read the `/root/.ssh/` directory and can read the `id_rsa` file, we can copy it to our machine and use the `-i` flag to log in with it:

```bash
lojique@htb[/htb]$ vim id_rsa
lojique@htb[/htb]$ chmod 600 id_rsa
lojique@htb[/htb]$ ssh user@10.10.10.10 -i id_rsa

root@remotehost#
```
