# Authentication Cracking

Comprehensive collection of passwords [here](https://wiki.skullsecurity.org/index.php/Passwords).

## Hydra

Fast network authentication cracker that supports different protocols such as:

* Cisco auth
* FTP
* HTTP
* IMAP
* RDP
* SMB
* SSH
* Telnet

Hydra can use dictionaries of usernames or passwords and can also perform pure brute force password attacks

Its architecture is based on modules which is a piece of code that lets Hydra attack a specific protocol

To get detailed information about a module, you can use the `-U` command line switch

You can check the available modules in the "supported services" section of the help

![](<../../../../.gitbook/assets/image (30) (1) (1) (1) (1) (1) (1) (1).png>)

To launch a dictionary attack against a service with a list of usernames (inside a users.txt file) and a list of passwords (pass.txt file):

```
hydra -L users.txt -P pass.txt <service://server> <options>
```

### Example: Telnet

```
hydra -L users.txt -P pass.txt telnet://target.server
```

### Example: HTTP Basic Auth Attack

```
hydra -L users.txt -P pass.txt http-get://localhost/
```

![](<../../../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1) (1).png>)
