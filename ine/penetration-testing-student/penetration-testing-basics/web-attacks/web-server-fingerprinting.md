# Web Server Fingerprinting

* Fingerprinting a web server means detecting:
  * The daemon providing the web server service
    * IIS, Apache, nginx, etc.
  * Its version
  * The OS of the machine hosting the server

## Fingerprinting with Netcat

* Known as the TCP/IP Swiss army knife
* Can be used both as a server or client
* You can fingerprint a web server using netcat as a client to manually send requests to the server (**banner grabbing**)
* Can only be used to connect to an HTTP web server

```
nc <target address> 80
```

* After connecting, send a valid HTTP request using the HEAD HTTP verb and hit return twice

```
HEAD / HTTP/1.0
```

![](<../../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1).png>)

## Fingerprinting with OpenSSL

* Used to establish a connection to an HTTPS service and send the usual HEAD HTTP Verb

```bash
openssl s_client -connect target.site:443
HEAD / HTTP/1.0
```

## Limits of Manual Fingerprinting

* sysadmins can customize web servers banners to make fingerprinting harder for attackers
* Automatic tools go beyond banner grabbing
  * They fingerprint web server by checking small implementation-dependent details such as:
    * Headers ordering in response messages
    * Errors handling

## Fingerprinting with Httprint

* Tool that uses a signature-based technique to identify web servers
* This is the most used syntax&#x20;

```
httprint -P0 -h <target hosts> -s <signature file>
// -P0 avoids pinging the host (most web servers do not respond to ping echo requestss
// -h lists of hosts (advised to use IPs)
// -s sets the signature file to use
```

![httprint example](<../../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1) (1).png>)
