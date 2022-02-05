# Active Directory

## Basics

Active Directory is a collection of machines and servers connected inside of domains, that are a collective part of a bigger forest of domains, that make up the Active Directory network. Active Directory contains many functioning bits and pieces, a majority of which we will be covering in the upcoming tasks

* **Domain Controllers**
* **Forests, Trees, Domains**
* **Users + Groups**&#x20;
* **Trusts**
* **Policies**&#x20;
* **Domain Services**

The majority of large companies use Active Directory because it allows for the control and monitoring of their user's computers through a single domain controller. It allows a single user to sign in to any computer on the active directory network and have access to his or her stored files and folders in the server, as well as the local storage on that machine. This allows for any user in the company to use any machine that the company owns, without having to set up multiple users on a machine. Active Directory does it all for you.

## Physical Active Directory

The physical Active Directory is the servers and machines on-premise, these can be anything from domain controllers and storage servers to domain user machines; everything needed for an Active Directory environment besides the software.\


### Domain Controllers

﻿A domain controller is a Windows server that has Active Directory Domain Services (AD DS) installed and has been promoted to a domain controller in the forest. Domain controllers are the center of Active Directory -- they control the rest of the domain.&#x20;

#### Tasks of a domain controller:&#x20;

* Holds the AD DS data store&#x20;
* Handles authentication and authorization services&#x20;
* Replicates updates from other domain controllers in the forest
* Allows admin access to manage domain resources

### AD DS Data Store&#x20;

The Active Directory Data Store holds the databases and processes needed to store and manage directory information such as users, groups, and services.&#x20;

#### Some of the contents and characteristics of the AD DS Data Store:

* Contains the NTDS.dit - a database that contains all of the information of an Active Directory domain controller as well as password hashes for domain users
* Stored by default in %SystemRoot%\NTDS
* Accessible only by the domain controller

## The Forest

The forest is what defines everything; it is the container that holds all of the other bits and pieces of the network together -- without the forest all of the other trees and domains would not be able to interact

![](https://i.imgur.com/EZawnqU.png)

﻿﻿A forest is a collection of one or more domain trees inside of an Active Directory network. It is what categorizes the parts of the network as a whole.

The Forest consists of:

* **Trees** - A hierarchy of domains in Active Directory Domain Services
* **Domains** - Used to group and manage objects&#x20;
* **Organizational** **Units (OUs)** - Containers for groups, computers, users, printers and other OUs
* **Trusts** - Allows users to access resources in other domains
* **Objects** - users, groups, printers, computers, shares
* **Domain Services** - DNS Server, LLMNR, IPv6
* **Domain** **Schema** - Rules for object creation

## Users & Groups

The users and groups that are inside of an Active Directory are up to you; when you create a domain controller it comes with default groups and two default users: Administrator and guest. It is up to you to create new users and create new groups to add users to.

### Users Overview

﻿There are four main types of users found in an Active Directory network; however, there can be more depending on how a company manages the permissions of its users. The four types are:&#x20;

* **Domain Admins** - This is the big boss: they control the domains and are the only ones with access to the domain controller.
* **Service Accounts** (Can be Domain Admins) - These are for the most part never used except for service maintenance, they are required by Windows for services such as SQL to pair a service with a service account
* **Local Administrators** - These users can make changes to local machines as an administrator and may even be able to control other normal users, but they cannot access the domain controller
* **Domain Users** - These are your everyday users. They can log in on the machines they have the authorization to access and may have local administrator rights to machines depending on the organization.

### Groups Overview

﻿Groups make it easier to give permissions to users and objects by organizing them into groups with specified permissions. There are two overarching types of Active Directory groups:&#x20;

* **Security Groups** - These groups are used to specify permissions for a large number of users
* **Distribution Groups** - These groups are used to specify email distribution lists. As an attacker these groups are less beneficial to us but can still be beneficial in enumeration

### Default Security Groups&#x20;

﻿There are a lot of default security groups. Here is a brief outline of the security groups:

* **Domain Controllers** - All domain controllers in the domain
* **Domain Guests** - All domain guests
* **Domain Users** - All domain users
* **Domain Computers** - All workstations and servers joined to the domain
* **Domain Admins** - Designated administrators of the domain
* **Enterprise Admins** - Designated administrators of the enterprise
* **Schema Admins** - Designated administrators of the schema
* **DNS Admins** - DNS Administrators Group
* **DNS Update Proxy** - DNS clients who are permitted to perform dynamic updates on behalf of some other clients (such as DHCP servers).
* **Allowed RODC Password Replication Group** - Members in this group can have their passwords replicated to all read-only domain controllers in the domain
* **Group Policy Creator Owners** - Members in this group can modify group policy for the domain
* **Denied RODC Password Replication Group** - Members in this group cannot have their passwords replicated to any read-only domain controllers in the domain
* **Protected Users** - Members of this group are afforded additional protections against authentication security threats. See http://go.microsoft.com/fwlink/?LinkId=298939 for more information.
* **Cert Publishers** - Members of this group are permitted to publish certificates to the directory
* **Read-Only Domain Controllers** - Members of this group are Read-Only Domain Controllers in the domain
* **Enterprise Read-Only Domain Controllers** - Members of this group are Read-Only Domain Controllers in the enterprise
* **Key Admins** - Members of this group can perform administrative actions on key objects within the domain.
* **Enterprise Key Admins** - Members of this group can perform administrative actions on key objects within the forest.
* **Cloneable Domain Controllers** - Members of this group that are domain controllers may be cloned.
* **RAS and IAS Servers** - Servers in this group can access remote access properties of users

## Trusts & Policies

Trusts and policies go hand in hand to help the domain and trees communicate with each other and maintain "security" inside of the network. They put the rules in place of how the domains inside of a forest can interact with each other, how an external forest can interact with the forest, and the overall domain rules or policies that a domain must follow.

### Domain Trusts Overview

﻿Trusts are a mechanism in place for users in the network to gain access to other resources in the domain. For the most part, trusts outline the way that the domains inside of a forest communicate to each other, in some environments trusts can be extended out to external domains and even forests in some cases.

![](https://i.imgur.com/4uGI3bF.png)

There are two types of trusts that determine how the domains communicate

* Directional - The direction of the trust flows from a trusting domain to a trusted domain
* Transitive - The trust relationship expands beyond just two domains to include other trusted domains

The type of trusts put in place determines how the domains and trees in a forest are able to communicate and send data to and from each other. When attacking an Active Directory environment you can sometimes abuse these trusts in order to move laterally throughout the network.&#x20;

### Domain Policies Overview&#x20;

Policies are a very big part of Active Directory, they dictate how the server operates and what rules it will and will not follow. You can think of domain policies like domain groups, except instead of permissions they contain rules, and instead of only applying to a group of users, the policies apply to a domain as a whole. They simply act as a rulebook for Active Directory that a domain admin can modify and alter as they deem necessary to keep the network running smoothly and securely. Along with the very long list of default domain policies, domain admins can choose to add in their own policies not already on the domain controller. For example: if you wanted to disable windows defender across all machines on the domain you could create a new group policy object to disable Windows Defender. The options for domain policies are almost endless and are a big factor for attackers when enumerating an Active Directory network.&#x20;

Here are a couple of the  many policies that are default or you can create in an Active Directory environment:&#x20;

* Disable Windows Defender - Disables windows defender across all machine on the domain
* Digitally Sign Communication (Always) - Can disable or enable SMB signing on the domain controller

\
