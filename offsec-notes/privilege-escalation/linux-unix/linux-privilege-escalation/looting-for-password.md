# Looting for password

## Files containing passwords

```bash
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```

## Old passwords in /etc/security/opasswd

The <mark style="color:green;">`/etc/security/opasswd`</mark> file is used also by pam\_cracklib to keep the history of old passwords so that the user will not reuse them.

{% hint style="danger" %}
Treat your opasswd file like your /etc/shadow file because it will end up containing user password hashes
{% endhint %}

## Last edited files

Files that were edited in the last 10 minutes

```
find / -mmin -10 2>/dev/null | grep -Ev "^/proc"
```

## In memory passwords

```
strings /dev/mem -n10 | grep -i PASS
```

## Find sensitive files

```
$ locate password | more           
/boot/grub/i386-pc/password.mod
/etc/pam.d/common-password
/etc/pam.d/gdm-password
/etc/pam.d/gdm-password.original
/lib/live/config/0031-root-password
...
```
