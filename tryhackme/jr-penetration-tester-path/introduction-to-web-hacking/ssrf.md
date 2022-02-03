---
description: >-
  Learn how to exploit Server-Side Request Forgery (SSRF) vulnerabilities,
  allowing you to access internal server resources.
---

# SSRF

## What is an SSRF?

Server-Side Request Forgery is a vulnerability that allows a malicious user to cause the webserver to make an additional or edited HTTP request to the resource of the attacker's choosing.

### Types of SSRF

There are two types of SSRF vulnerability; the first is a regular SSRF where data is returned to the attacker's screen. The second is a Blind SSRF vulnerability where an SSRF occurs, but no information is returned to the attacker's screen.&#x20;

### What's the impact?

A successful SSRF attack can result in any of the following:

* Access to unauthorized areas
* Access to customer/organizational data
* Ability to Scale to internal networks
* Reveal authentication tokens/credentials

## SSRF Examples

This [video ](https://www.youtube.com/watch?v=zWGOJSnH0HI)is excellent in helping to explain this room

![](../../../.gitbook/assets/image.png)

![](<../../../.gitbook/assets/image (23) (1) (1) (1) (1) (1).png>)

![](<../../../.gitbook/assets/image (32) (1) (1) (1) (1).png>)

![](<../../../.gitbook/assets/image (24) (1) (1) (1).png>)

### Practical

![](<../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1).png>)

We use the following code which tells us where we want to go and stops the remaining path from being appended to the end of our URL and instead turns it into a parameter (?x=) on the query string by using <mark style="color:red;">`&x=`</mark>

```
server.website.thm/flag?id=9&x=
```

![](<../../../.gitbook/assets/image (31) (1) (1) (1) (1).png>)

## Finding an SSRF

Potential SSRF vulnerabilities can be spotted in web applications in many different ways. Here is an example of four common places to look:

When a full URL is used in a parameter in the address bar:

![](<../../../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1).png>)

A hidden field in a form:

![](<../../../.gitbook/assets/image (1) (1).png>)

A partial URL such as just the hostname:

![](<../../../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

Or perhaps only the path of the URL:

![](<../../../.gitbook/assets/image (20) (1) (1) (1) (1) (1).png>)

Some of these examples are easier to exploit than others, and this is where a lot of trial and error will be required to find a working payload.

If working with a blind SSRF where no output is reflected back to you, you'll need to use an external HTTP logging tool to monitor requests such as requestbin.com, your own HTTP server or Burp Suite's Collaborator client.

## Defeating Common SSRF Defenses

More security-savvy developers aware of the risks of SSRF vulnerabilities may implement checks in their applications to make sure the requested resource meets specific rules. There are usually two approaches to this, either a deny list or an allow list.

### Deny List

A Deny List is where all requests are accepted apart from resources specified in a list or matching a particular pattern. A Web Application may employ a deny list to protect sensitive endpoints, IP addresses or domains from being accessed by the public while still allowing access to other locations. A specific endpoint to restrict access is the localhost, which may contain server performance data or further sensitive information, so domain names such as localhost and 127.0.0.1 would appear on a deny list. Attackers can bypass a Deny List by using alternative localhost references such as 0, 0.0.0.0, 0000, 127.1, 127._._.\*, 2130706433, 017700000001 or subdomains that have a DNS record which resolves to the IP Address 127.0.0.1 such as 127.0.0.1.nip.io.

Also, in a cloud environment, it would be beneficial to block access to the IP address 169.254.169.254, which contains metadata for the deployed cloud server, including possibly sensitive information. An attacker can bypass this by registering a subdomain on their own domain with a DNS record that points to the IP Address 169.254.169.254.

### Allow List

An allow list is where all requests get denied unless they appear on a list or match a particular pattern, such as a rule that an URL used in a parameter must begin with https://website.thm. An attacker could quickly circumvent this rule by creating a subdomain on an attacker's domain name, such as https://website.thm.attackers-domain.thm. The application logic would now allow this input and let an attacker control the internal HTTP request.

### Open Redirect

If the above bypasses do not work, there is one more trick up the attacker's sleeve, the open redirect. An open redirect is an endpoint on the server where the website visitor gets automatically redirected to another website address. Take, for example, the link https://website.thm/link?url=https://tryhackme.com. This endpoint was created to record the number of times visitors have clicked on this link for advertising/marketing purposes. But imagine there was a potential SSRF vulnerability with stringent rules which only allowed URLs beginning with https://website.thm/. An attacker could utilize the above feature to redirect the internal HTTP request to a domain of the attacker's choice.

## SSRF Practical

We've come across two new endpoints during a content discovery exercise against the Acme IT Support website. The first one is /private, which gives us an error message explaining that the contents cannot be viewed from our IP address. The second is a new version of the customer account page at `/customers/new-account-page` with a new feature allowing customers to choose an avatar for their account.

First, create a customer account and sign in. Once you've signed in, visit `https://10-10-124-231.p.thmlabs.com/customers/new-account-page` to view the new avatar selection feature. By viewing the page source of the avatar form, you'll see the avatar form field value contains the path to the image. The background-image style can confirm this in the above DIV element as per the screenshot below:

![](<../../../.gitbook/assets/image (3) (1).png>)

If you choose one of the avatars and then click the Update Avatar button, you'll see the form change and, above it, display your currently selected avatar. Viewing the page source will show your current avatar is displayed using the data URI scheme, and the image content is base64 encoded as per the screenshot below.

![](<../../../.gitbook/assets/image (29) (1) (1) (1) (1) (1) (1).png>)

Now let's try making the request again but changing the avatar value to private in hopes that the server will access the resource and get past the IP address block. To do this, firstly, right-click on one of the radio buttons on the avatar form and select Inspect:

![](<../../../.gitbook/assets/image (30) (1) (1) (1) (1).png>)

And then edit the value of the radio button to private:

![](<../../../.gitbook/assets/image (33) (1) (1) (1) (1) (1).png>)

And then click the Update Avatar button. Unfortunately, it looks like the web application has a deny list in place and has blocked access to the /private endpoint.

![](<../../../.gitbook/assets/image (18) (1) (1) (1) (1).png>)

As you can see from the error message, the path cannot start with **/private** but don't worry, we've still got a trick up our sleeve to bypass this rule. We can use a directory traversal trick to reach our desired endpoint. Try setting the avatar value to `x/../private`

![](<../../../.gitbook/assets/image (8) (1) (1).png>)

You'll see we have now bypassed the rule, and the user updated the avatar. This trick works because when the web server receives the request for `x/../private`, it knows that the ../ string means to move up a directory that now translates the request to just /private.

Viewing the page source of the avatar form, you'll see the currently set avatar now contains the contents from the /private directory in base64 encoding, decode this content and it will reveal a flag that you can enter below.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

Information courtesy of tryhackme

Go to the room [here](https://tryhackme.com/room/ssrfqi)
