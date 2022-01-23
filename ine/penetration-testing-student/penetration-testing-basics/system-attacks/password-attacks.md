# Password Attacks

## John the Ripper

### Brute Force

For this scenario, let's say we've got unauthorized access to a Linux machine and made a copy of the password files:

* `/etc/passwd` (contains information about user accounts)
* `/etc/shadow` (contains the actual password hashes)

John needs the username and password hashes to be in the same file so use the following command:

```
unshadow passwd shadow > file.txt
```

Usually a password file contains passwords of multiple users. To crack just some of them, you can use the `-users` option

```
john -incremental -users:<users list> <file to crack>
```

![](<../../../../.gitbook/assets/image (28) (1).png>)

You can use `--show` to display the passwords recovered by John

![](<../../../../.gitbook/assets/image (21) (1) (1).png>)

### Dictionary Attack

We can run dictionary attacks with John by using the `-wordlist` argument

```
john -wordlist<=custom wordlist file> <file to crack>
```

Some mangling can be applied by using the `-rules` parameter

Mangling rules help with case differences or some digits at the end of a word

For example, variations on "cat" could be: cat12, caT, CAT, Cat, CAt, c@t and so on...

![In this example, the crackme file was cracked using the John default wordlist](<../../../../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/image (22) (1) (1) (1).png>)

![dictionary mangling enabled](<../../../../.gitbook/assets/image (27) (1) (1).png>)

## Rainbow Tables

A rainbow table is a table containing links between the results of a run of one hashing function and another

The space needed to store a rainbow table depends on how many characters are allowed in a password and its length. The specific hash function used to store the password also plays a role

Rainbow tables' sizes vary from hundreds of megabytes to hundreds of gigabytes

A lookup in the rainbow tables will save cracking tools the computational time needed to hash every candidate password

A limit of rainbow tables is the storage space needed to guarantee successful cracking sessions

### Ophcrack

Rainbow cracking tool aimed at Windows password recovery&#x20;

Can run on Windows, Linux, Unix and OSX
