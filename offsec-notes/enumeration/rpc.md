# RPC

```
# anonymous auth
rpcclient -U "" 10.10.10.10
rpcclient -U "" -N 10.10.10.161

enumdomusers
enumprinters
enumdomgroups

# grep users for password spraying and ASEProasting
rpcclient -U "" <ip> -N -c "enumdomusers" | grep -oP '\[.*?\]' | grep "0x" -v | tr -d '[]' > userlist.txt
```
