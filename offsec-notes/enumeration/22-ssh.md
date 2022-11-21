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
sudo nmap -p22 --script ssh2-enum-algos IP
sudo nmap -p22 --script ssh-hostkey --script-args ssh_hostkey=full IP
sudo nmap -p22 --script ssh-auth-methods --script-args="ssh.user=root" IP
```
