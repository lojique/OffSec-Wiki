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

