# NFS

```bash
# show available NFS shares
showmount -e 10.129.14.128

# Mounting NFS share
mkdir nfs_share
sudo mount -t nfs 10.129.14.128:/ ./nfs_share/ -o nolock
# sudo mount -t nfs $IP:/remote-nfs-name nfs_share/

cd target-NFS
tree .
ls -l mnt/nfs/

# Authenticated mount
sudo mount -0 username=admin,password='P@ssw0rd123!' -t cifs \\\\10.10.10.10\\share /mnt

# unmount
sudo umount ./target-NFS
```

| Tag      | Function                                                                     |
| -------- | ---------------------------------------------------------------------------- |
| mount    | Execute the mount command                                                    |
| -t nfs   | Type of device to mount, then specifying that it's NFS                       |
| IP:share | The IP Address of the NFS server, and the name of the share we wish to mount |
| -nolock  | Specifies not to use NLM locking                                             |
