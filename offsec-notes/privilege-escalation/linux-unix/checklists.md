# Checklists

### Tools

There are many scripts that you can execute on a linux machine which automatically enumerate system information, processes, and files to locate privilege escalation vectors. Here are a few:

*   [LinPEAS - Linux Privilege Escalation Awesome Script](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)

    ```bash
    wget "https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh" -O linpeas.sh
    curl "https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh" -o linpeas.sh
    ./linpeas.sh -a #all checks - deeper system enumeration, but it takes longer to complete.
    ./linpeas.sh -s #superfast & stealth - This will bypass some time consuming checks. In stealth mode Nothing will be written to the disk.
    ./linpeas.sh -P #Password - Pass a password that will be used with sudo -l and bruteforcing other users
    ```
*   [LinuxSmartEnumeration - Linux enumeration tools for pentesting and CTFs](https://github.com/diego-treitos/linux-smart-enumeration)

    ```bash
    wget "https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh" -O lse.sh
    curl "https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh" -o lse.sh
    ./lse.sh -l1 # shows interesting information that should help you to privesc
    ./lse.sh -l2 # dump all the information it gathers about the system
    ```
*   [LinEnum - Scripted Local Linux Enumeration & Privilege Escalation Checks](https://github.com/rebootuser/LinEnum)

    ```bash
    ./LinEnum.sh -s -k keyword -r report -e /tmp/ -t
    ```
*   [linux-exploit-suggester](https://github.com/The-Z-Labs/linux-exploit-suggester)\


    ```bash
    wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh
    ```
* [linuxprivchecker.py - a Linux Privilege Escalation Check Script](https://github.com/sleventyeleven/linuxprivchecker)
* [BeRoot - Privilege Escalation Project - Windows / Linux / Mac](https://github.com/AlessandroZ/BeRoot)
* [unix-privesc-check - Automatically exported from code.google.com/p/unix-privesc-check](https://github.com/pentestmonkey/unix-privesc-check)
* [Privilege Escalation through sudo - Linux](https://github.com/TH3xACE/SUDO\_KILLER)

### Checklists

* Kernel and distribution release details
* System Information:
  * Hostname
  * Networking details:
  * Current IP
  * Default route details
  * DNS server information
* User Information:
  * Current user details
  * Last logged on users
  * Shows users logged onto the host
  * List all users including uid/gid information
  * List root accounts
  * Extracts password policies and hash storage method information
  * Checks umask value
  * Checks if password hashes are stored in /etc/passwd
  * Extract full details for 'default' uid's such as 0, 1000, 1001 etc
  * Attempt to read restricted files i.e. /etc/shadow
  * List current users history files (i.e .bash\_history, .nano\_history, .mysql\_history , etc.)
  * Basic SSH checks
* Privileged access:
  * Which users have recently used sudo
  * Determine if /etc/sudoers is accessible
  * Determine if the current user has Sudo access without a password
  * Are known 'good' breakout binaries available via Sudo (i.e. nmap, vim etc.)
  * Is root's home directory accessible
  * List permissions for /home/
* Environmental:
  * Display current $PATH
  * Displays env information
* Jobs/Tasks:
  * List all cron jobs
  * Locate all world-writable cron jobs
  * Locate cron jobs owned by other users of the system
  * List the active and inactive systemd timers
* Services:
  * List network connections (TCP & UDP)
  * List running processes
  * Lookup and list process binaries and associated permissions
  * List inetd.conf/xined.conf contents and associated binary file permissions
  * List init.d binary permissions
* Version Information (of the following):
  * Sudo
  * MYSQL
  * Postgres
  * Apache
    * Checks user config
    * Shows enabled modules
    * Checks for htpasswd files
    * View www directories
* Default/Weak Credentials:
  * Checks for default/weak Postgres accounts
  * Checks for default/weak MYSQL accounts
* Searches:
  * Locate all SUID/GUID files
  * Locate all world-writable SUID/GUID files
  * Locate all SUID/GUID files owned by root
  * Locate 'interesting' SUID/GUID files (i.e. nmap, vim etc)
  * Locate files with POSIX capabilities
  * List all world-writable files
  * Find/list all accessible \*.plan files and display contents
  * Find/list all accessible \*.rhosts files and display contents
  * Show NFS server details
  * Locate \*.conf and \*.log files containing keyword supplied at script runtime
  * List all \*.conf files located in /etc
  * Locate mail
* Platform/software specific tests:
  * Checks to determine if we're in a Docker container
  * Checks to see if the host has Docker installed
  * Checks to determine if we're in an LXC container
