# SMTP

```
nc -nv 10.11.1.217 25
```

```
rpcinfo -s 10.11.1.72
rpcinfo -p 10.11.1.72
```

```python
#!/usr/bin/python
# opens a TCP socket, connects to the SMTP server, 
# and issues a VRFY command for a given username

import socket
import sys

if len(sys.argv) != 2:
        print "Usage: vrfy.py <username>"
        sys.exit(0)

# Create a Socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to the Server
connect = s.connect(('10.11.1.217',25))

# Receive the banner
banner = s.recv(1024)

print banner

# VRFY a user
s.send('VRFY ' + sys.argv[1] + '\r\n')
result = s.recv(1024)

print result

# Close the socket
s.close()
```
