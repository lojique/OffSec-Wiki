# 22 - SSH

#### banner grabbing

```
nc -nv IP 22
telnet IP 22
```

#### ssh-audit

```
ssh-audit IP/Domain
```

#### sshscan

```
./sshscan.py -t IP
```

#### public ssh key of server

```
ssh-keyscan -t rsa IP/Domain
```

#### nmap scripts

```python
sudo nmap -p22 40.143.62.144 --script ssh2-enum-algos
sudo nmap -p22 40.143.62.144 --script ssh-hostkey --script-args ssh_hostkey=full
sudo nmap -p22 40.143.62.144 --script ssh-auth-methods --script-args="ssh.user=root"
```
