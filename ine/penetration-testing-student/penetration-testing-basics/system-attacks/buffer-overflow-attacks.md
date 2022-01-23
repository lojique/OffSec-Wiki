# Buffer Overflow Attacks

## BOF Attacks

Many different attacks widely exploit buffer overflow vulnerabilities which work by taking control of the execution flow of a piece of software or a routine of the operating system

A buffer overflow attack can lead to:

* An application or OS crash, thus causing a DoS
* Privilege escalation
* Remote code execution
* Security features bypass

## Buffers

A buffer is an area in the computer RAM reserved for temporary data storage such as:

* User input
* Parts of a video file
* Server banners received by a client application
* and so on

Buffers have finite size; they can only contain a certain amount of data

If a client-server app is designed to accept only 8-characters long usernames, the username buffer will by 8 bytes long

![](<../../../../.gitbook/assets/image (14) (1).png>)

If the developer of an app does not enforce buffers limits, an attacker could find a way to write data beyond those limits, thus actually writing arbitrary code in the computer RAM; this can be exploited to get control over the program execution flow

### Example

![](<../../../../.gitbook/assets/image (12) (1).png>)

![](<../../../../.gitbook/assets/image (10) (1).png>)

![](<../../../../.gitbook/assets/image (33) (1) (1) (1).png>)

## The Stack

Buffers are stored in a special data structure in the computer memory called a stack

A **stack** is a data structure used to store data

You can only add something on the stack or remove something from the top of the stack

* This is called **Last in First Out (LIFO)** and uses two methods:
  * **Push**, which adds an element to the stack
  * **Pop** removes the last inserted element

In modern operating systems, the stack is used in a more flexible way

Even if push and pop are still used, an application can randomly access a position on the stack to read and write data

To save some stack space for later use, the application can simply reserve some memory allocations on the stack and then access them

### Example

![](<../../../../.gitbook/assets/image (26) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/image (16) (1) (1).png>)

### Overflows in the Stack

![](<../../../../.gitbook/assets/image (31) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/image (13).png>)

![](<../../../../.gitbook/assets/image (6) (1).png>)

## The Stack in an Application

The stack used by applications and operating systems does not only contain data, but also information about the execution flow

While analyzing a function call, you can see that the stack contains the function parameters, its local variables and the memory address where the execution of the program must continue after the function returns

So overwriting a function return address means getting control over the application

In addition, if an attacker manages to write some valid code in RAM, they can force the victim function to run their code

A raw overflow that just overwrites some memory locations will crash the app, while a well-engineered attack is able to execute code on the victim machine

## How Buffer Overflow Attacks Work

We should understand that if an attacker manages to overflow Local Variable 1, they are able to overwrite Base Pointer and then Return Address

![](<../../../../.gitbook/assets/image (15) (1) (1).png>)

If they overwrite Return Address with the right value, they are able to control the execution flow of the program

This technique can be exploited by writing custom tools and applications or by using hacking tools such as Metasploit
