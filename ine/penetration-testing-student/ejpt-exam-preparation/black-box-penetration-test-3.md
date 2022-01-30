# Black-box Penetration Test 3

## Scenario

* You have been engaged in a Black-box Penetration Test (172.16.37.0/24 range). Your goal is to read the flag file on each machine. On some of them, you will be required to exploit a remote code execution vulnerability in order to read the flag.
* Some machines are exploitable instantly but some might require exploiting other ones first. Enumerate every compromised machine to identify valuable information, that will help you proceed further into the environment.
* If you are stuck on one of the machines, don't overthink and start pentesting another one.
*   When you read the flag file, you can be sure that the machine was successfully compromised. But keep your eyes open - apart from the flag, other useful information may be present on the system.

    > This is not a CTF! The flags' purpose is to help you identify if you fully compromised a machine or not.
    >
    > The solutions contain the shortest path to compromise each machine. You should follow the penetration testing process covered in its entirety!

## Goals

* Discover and exploit all machines on the network
* Read all flag files (one per machine)
* Obtain root privileges on both machines (meterpreter's autoroute functionality and ncrack's minimal.usr list will prove useful)

## What you will learn

* Network discovery
* Pivoting to other networks
* Basic privilege escalation

## Recommended tools

* Dirb
* Metasploit framework (recommended version 5)
* Nmap
* FTP Utility

## Stuff I need to write down

