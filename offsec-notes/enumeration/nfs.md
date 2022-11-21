# NFS

```
ls -1 /usr/share/nmap/scripts/nfs*
nmap -p 111 --script nfs* 10.11.1.72
```

```
nmap -v -p 111 10.11.1.1-254
# find services possibly registered with rpcbind
nmap -sV -p 111 --script=rpcinfo 10.11.1.1-254 
```

```
sudo mount -t nfs IP:share /tmp/mount/ -nolock
```

| Tag      | Function                                                                     |
| -------- | ---------------------------------------------------------------------------- |
| mount    | Execute the mount command                                                    |
| -t nfs   | Type of device to mount, then specifying that it's NFS                       |
| IP:share | The IP Address of the NFS server, and the name of the share we wish to mount |
| -nolock  | Specifies not to use NLM locking                                             |

### Authenticated mount

```
sudo mount -0 username=admin,password='P@ssw0rd123!' -t cifs \\\\10.10.10.10\\share
```
