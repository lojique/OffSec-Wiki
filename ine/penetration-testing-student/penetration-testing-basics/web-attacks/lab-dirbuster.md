---
description: >-
  In this lab, you will learn to perform Directory Enumeration attack using the
  Dirbuster tool.
---

# Lab - Dirbuster

## Lab Environment

In this lab environment, the user is going to get access to a Kali GUI instance. An instance of the Mutillidae web application can be accessed using the tools installed on Kali at http://demo.ine.local

**Objective:** Perform directory enumeration with Dirbuster!

![0](https://assets.ine.com/content/pta-labs/10\_dirbuster/0.png)

## Instructions

* **Use Directory wordlist:** /usr/share/wordlists/dirb/common.txt

## Tools

The best tools for this lab are:

* Dirbuster
* A web browser

## My Solution

First we will ping the web app just to make sure we can reach it

![](<../../../../.gitbook/assets/image (24) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Next, we'll run an nmap scan on the target to gather some info

![](<../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png>)

We know an Apache HTTP web server is running on port 80 with the version of 2.4.7

Don't forget about checking out the website itself!

![](<../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png>)

Now we'll set up Dirbuster

We've been told which wordlist to use

I used these extensions so I get try to get the most valuable information returned

![](<../../../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

We'll let this run for a while and also increase the number of running threads to get results faster

And after waiting for a while we check the 'Results - Tree View' tab and see something interesting

![](<../../../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Checking out the file reveals all username and plain-text passwords stored on here

![](<../../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png>)
