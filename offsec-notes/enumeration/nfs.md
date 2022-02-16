# NFS

```
sudo mount -t nfs IP:share /tmp/mount/ -nolock
```

| Tag      | Function                                                                     |
| -------- | ---------------------------------------------------------------------------- |
| mount    | Execute the mount command                                                    |
| -t nfs   | Type of device to mount, then specifying that it's NFS                       |
| IP:share | The IP Address of the NFS server, and the name of the share we wish to mount |
| -nolock  | Specifies not to use NLM locking                                             |

\
