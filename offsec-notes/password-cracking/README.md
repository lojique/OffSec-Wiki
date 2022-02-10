# Password Cracking

Hashes found in web app databases are more likely to be MD5

Windows passwords are hashed using NTLM (variant of MD4)

On Linux, password hashes are stored in /etc/shadow

On Windows, password hashes (split into NT and LM hashes are stored in the Security Account Manager (SAM)

{% hint style="info" %}
Mimimatz can be used to dump Windows hashes
{% endhint %}

## Common Unix-Style Password Prefixes

| Prefix                      | Algorithm                                                  |
| --------------------------- | ---------------------------------------------------------- |
| $1$                         | md5crypt, used in Cisco stuff and older Linux/Unix systems |
| $2$, $2a$, $2b$, $2x$, $2y$ | Bcrypt (Popular for web applications)                      |
| $6$                         | sha512crypt (Default for most Linux/Unix systems)          |

