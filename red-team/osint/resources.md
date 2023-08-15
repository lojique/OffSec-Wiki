# Resources

## Company Information

### Locations

Find `countries`, `cities`, `addresses`, and possibly even `maps` that show the company's locations

Social networks are used to share much information, especially information that potential customers can use to learn more about the company. This includes the publication of locations of offices or production facilities. This type of information can be found on many social networks such as [Twitter](https://twitter.com/), [LinkedIn](https://www.linkedin.com/), [Instagram](https://www.instagram.com/), [Snapchat](https://www.snapchat.com/), [YouTube](https://www.youtube.com/), [WeChat](https://www.wechat.com/en/), [Facebook](https://www.facebook.com/), and others.

### Staff

Most of the time, we can find the company's essential contact persons on the "`About Us`" page

Linkedin

We can use `search engines` we know from which we will discover other employees, the social networks on which they appear, the `development platforms` used by developers, and other files from which we can extract metadata. We can obtain further information through these `internal leaks`.

If files are found, we can use exiftool to extract metadata

{% embed url="https://www.hackers-arise.com/post/2019/05/28/osint-part-3-extracting-employee-names-from-companies-tesla-and-breitbart-on-linkedin" %}

### Contact Information

If the company wants to present itself and clarify individual needs in the best possible way, the `contact details` page on their website will list people from different departments.

We can use social networks to find `contacts`, `partners`, `co-developers`, their `phone numbers`, and `email addresses`.

Find out the email addresses of the target domain with Hunter.io

{% embed url="https://hunter.io/" %}

We can verify each email address via [Emailrep.io](https://emailrep.io/) and check how the mail servers handle it.

[RocketReach](https://rocketreach.co/) allows us to identify email address formats by domain and the other email addresses linked to the corresponding managers. It is a paid service provider, but the results we get from it are very accurate, and it is an excellent way to find out links to the company's employees and the platforms they use.

{% embed url="https://allpeople.com/" %}

{% embed url="https://web.archive.org/" %}

### Business Records

Financial records, files, etc.

Using Google search, we can find and filter out files like .docx, PDFs, and others. We can use `Google Dorks` for these purposes. A small list of the most important "dorks" can be found [here](https://securitytrails.com/blog/google-hacking-techniques).

On [Crunchbase.com](https://www.crunchbase.com/), we can find some of the `published records` concerning our target company. Often we can find financial reports from which we can find more helpful information. These reports also list some information about employees.

### Services

We should understand the company's goals surrounding its services for its customers and itself.

Information about the target company's `partners` can also be found `in files`. This is because not all partners and technologies are presented and disclosed directly. However, these companies may be marked as "`Powered by`" in these files.

In addition to these files, information about services is often stored in detail in documents such as PDF files. This is often done because of the possibility to design these documents to be very visually appealing while communicating a lot of detailed information about the service. Here, too, appearance plays a significant role for the companies. After all, they always want to show that they are up to date and offer unique solutions for their customers' needs. It is precisely these self-developed solutions that provide us a potential attack vector that we could exploit.

Check social media and forums.

A wide variety of technical discussions and news are discussed in forums. These are very useful for us when we want to find out something about a company's performance and services. Every post in the forum usually has a `date` when we find out when an enhancement or application, or service was released. Based on this time-lapse, we can roughly estimate how up-to-date specific `software` may be.

### Social Networks

blogs, news, social media, wikis, documents & files, images & videos

On these social media platforms, such as [Twitter](https://twitter.com/), [LinkedIn](https://www.linkedin.com/), [Xing](https://www.xing.com/), [Facebook](https://www.facebook.com/), [Instagram](https://www.instagram.com/), [YouTube](https://www.youtube.com/), and others, we find not only the `latest news` and `developments` of the company but also older posts that can give us information about the `structure and technologies`.

Another often overlooked component, which does not directly belong to the social media category but fulfills the same purpose, is `newsletters`.

reddit, github, stackoverflow

## Infrastructure

### Domain Information

We often find references to `domains` or `subdomains` on social media platforms.

One of the most efficient methods of searching for `domains` is offered by various `search engines`

Another excellent way to find out information about our target domain is to use a particular `Search Engine Optimization` (`SEO`) field called `backlinks`. A backlink is a link from an external page to another website

Development platforms also offer excellent information resources for us here, as we will often find code that sometimes even provide information that is dangerous for the company. This information can range from `IP addresses` and `hostnames`, configuration files to `credentials`. These platforms include, but are not limited to:

<table data-header-hidden><thead><tr><th width="158"></th><th width="139"></th><th width="184"></th><th width="163"></th></tr></thead><tbody><tr><td><a href="https://github.com/">Github</a></td><td><a href="https://gitlab.com/">GitLab</a></td><td><a href="https://code.google.com/">Google Code</a></td><td><a href="https://bitbucket.org/">Bitbucket</a></td></tr></tbody></table>

Another source that searches against many different developer platforms is [Searchcode](https://searchcode.com/). It searches all possible codes for terms that we specify in the search and shows us the sources accordingly

{% embed url="https://app.neilpatel.com/en/seo_analyzer/backlinks" %}

### Public Domain Records

After we have identified and filtered out all the domains that belong to them, we can focus on their structure and composition

we need to find out at least four components:

<table data-header-hidden><thead><tr><th width="241"></th><th width="101"></th><th width="177"></th><th></th></tr></thead><tbody><tr><td><code>1. Netblocks / CIDR</code></td><td><code>2. ASN</code></td><td><code>3. DNS Servers</code></td><td><code>4. Mail Servers</code></td></tr></tbody></table>

1. **Netblocks / CIDR**

During a black-box penetration test, our client will often only provide the domain name. We can use this to find out a lot of helpful information. First of all, we should find the `IP address` of the main webserver(s) and the `IP address range(s)` / `CIDR`.

```
host www.domain.com
```

Now that we have the IP address of the web server, we can find out which `IP address range` it is located in. By default, we can work with the `Whois` protocol used by a distributed database system to retrieve information about internet domains and IP addresses and their owners.

```
whois [IP Address]
```

We can also search the `WHOIS` databases for the company name. We are often given different identification numbers (`mnt-ref`), which we can use for further searches. These stand for the maintainer objects, which are used as a reference for the organization objects.

```
whois -B --sources RIPE,ARIN target-company
```

We can also query the individual databases and identify the associated netblocks.

```
whois -h whois.arin.net target-company | grep -v "#" | sed -r '/^\s*$/d'
```

Here is the [ARIN list](https://www.arin.net/resources/registry/whois/rws/cli/) of other flags we can use to get more information from the maintainer objects.

**2. ASN**

We can also see another crucial piece of information here, the `OrigisAS`. This is the `Autonomous System Number` (`ASN`). This number is unique and is made publicly available so routing information can be exchanged with other systems. This is done with specific IP prefixes, which we can use to find out the netblocks (`public ASNs`). There are also `private ASNs` intended for systems that only communicate via a provider. Other protocols such as the `Border Gateway Protocol` (`BGP`) are used if this is the case. We can use [MXToolbox](https://mxtoolbox.com/SuperTool.aspx) and its `ASN Lookup` option with the ASN to find out how many subnets the owner has.

Another way to get more information about the domain is to use the [ICANN lookup](https://lookup.icann.org/lookup). Each domain is registered to an organization or person with a unique ID, which provides information about when the domain was created and expires.



**3. DNS Servers**

```
dig any domain.com
```

{% embed url="https://dnsdumpster.com/" %}

**4. Mail Servers**

Mail Server / Exchange server (`MX`) is one of the most critical services today, as it ensures that our emails reach the desired communication partners. Other servers use it as an interstation (relay) for sending spam or viruses. It often happens that `MX` servers do not only use specially registered servers as relays. This gives us the possibility to perform an `Open Relay Attack`.

[MXtoolbox](https://mxtoolbox.com/) offers an excellent service to test the `MX` servers for Open Relay.



Apart from already highly sensitive data such as user names and passwords, we can also find `configuration files` containing the latest administration settings. We can use [Searchcode](https://searchcode.com/) again with specific strings and known `IP addresses` or `domain names` to get such results.&#x20;

### Domain Structure

#### Subdomains

We can use a tool called [CTFR](https://github.com/UnaPibaGeek/ctfr). It uses `Certificate Transparency` logs from [Certificate Transparency](https://www.certificate-transparency.org/) and [crt.sh](https://crt.sh/)

```
./ctfr.py -d domain.com | grep -v "[-]"
```

We can now use [CTFR](https://github.com/UnaPibaGeek/ctfr) for each subdomain in a `For-Loop` because there may be other subdomains in the respective subdomains. After we have collected and documented all passive results, we can use a simple `For-Loop` in Bash to determine the corresponding IP address for each subdomain.

```shell-session
for sub in $(cat subdomains);do host $sub | grep "has" | cut -d" " -f1,4;done
```

Then we can use `whois` and `ipcalc` to find out the IP ranges for the respective IPv4 addresses.

Another great way to quickly search for subdomains is [C99.nl](https://subdomainfinder.c99.nl/index.php)

#### IP Addresses

_**Shodan**_

```
shodan domain <TARGET-DOMAIN> | grep -w "A" | cut -d"A" -f2 | cut -d" " -f7 | sort -u > IPv4s.txt
for ip in $(cat IPv4s.txt);do shodan host $ip;done
```

_**Spyse**_

[Spyse](https://spyse.com/) offers excellent results in an excellent presentation. Apart from the different search options this platform provides, the results are quite accurate and informative. For example, we can search for more IPv4 addresses, and we will find that `Spyse` results in IPv4 addresses that we have not found before.

After this, we can use [IPinfo.io](https://ipinfo.io/). This resource provides an excellent way to identify the subnets and hosting providers. Furthermore, we can use Spyse to search for additional subdomains by entering the top and second-level domains (e.g., target-company.htb).

_**Virtual Hosts**_

With the IP addresses and subdomains, we can determine how many and which subdomains are `virtual hosts` (`vHosts`). Some good sources that we can use are [Pentester-Tools](https://pentest-tools.com/information-gathering/find-virtual-hosts) and [Hacker-Target](https://hackertarget.com/).

We can easily recognize `vHosts`, because they share the same IP address for at least two subdomains

#### Development Platforms

For the developer platforms, we should be on the lookout for all possible files, as eventually, any of them may contain hints about `domain names` or `IP addresses`. Configuration files are of particular interest, as they may contain access data and have fixed IP addresses and (sub)domains. For this, we can again use [Searchcode](https://searchcode.com/) to find files of this kind quickly.

Another very interesting source of information is the [SEOptimer](https://www.seoptimer.com/). This is an SEO analysis tool that examines and evaluates the entire website for individual components

#### Leak Resources

Rapid7 started [Project Sonar](https://opendata.rapid7.com/sonar.fdns\_v2/), which collects and stores responses forwarding DNS requests. DNS records such as `A`, `AAAA`, `CNAME`, and `TXT` lookups are stored at certain intervals in individual `GZIP` files in the form of `JSON`. These databases are extensive and can exceed 30GB in compressed format. They contain (sub)domains, record types, and the corresponding IP addresses.&#x20;

```
pigz -dc rapid7-fdns-db.json.gz | grep "target-company"
```

### Cloud Storage

The tool [ip2provider.py](https://github.com/oldrho/ip2provider) will automatically compare the IP addresses with the netblocks and show if they are successful matches.

Installation & usage

```shell-session
git clone https://github.com/oldrho/ip2provider.git
cd ip2provider
pip3 install -r requirements.txt

cat Target_Company.IPv4s | ./ip2provider.py
```

{% embed url="https://buckets.grayhatwarfare.com/" %}

#### Development Platforms

Again, developer platforms are of great use to us, as we can search code to find out if specific files exist in connection with the cloud buckets. As we know, `Searchcode` also redirects us to the resource that points to the corresponding file. Often used strings in these files are `AccountName` and `AccountKey`, used for authorization (like username and password).



#### Leak Resources

Another source that can point us to the buckets and forward them is the aforementioned [Rapid7 database](https://opendata.rapid7.com/sonar.fdns\_v2/). Since this works with the forward DNS requests and responses and documents those, it is even very likely that we will find entries that will show us the corresponding buckets.

```
pigz -dc rapid7-fdns-db.json.gz | grep "target-company" | grep "core.microsoft.net\|s3.amazonaws.com\|s3-*.amazonaws.com\|googleapis.com/storage/v1/b/"
```

### Email Addresses

The company's website could have emails on the contact or about page

In the search engines, such as Google, we  use the corresponding Google dorks (`inurl:` & `intext:`) to optimize social networks' results.&#x20;

```
intext:"@domain.com" inurl:twitter.com
```

Another critical role for email addresses is their `reputation`. This could tell us whether the corresponding email address has already been used for spam activities, has been blacklisted, whether `SPF` or `DMARC` is used, and much more. To get this information, we can use the information resource called [Emailrep](https://emailrep.io/). Apart from the information we get from it, using the API is very useful for us as we can easily use it with `cURL`.

```
curl -s emailrep.io/info@target-company.com
```

[theHarvester](https://github.com/laramies/theHarvester). This tool searches the information resources we provide, such as Google, Netcraft, Spyse, Twitter, and others for entries. Some of these services offer API keys that are bound to the corresponding account to execute the requests.

```
python3 theHarvester.py -d domain.com -b all
```

#### Leak Resources

Email addresses often play a significant role in this field, as they are linked via a wide variety of platforms. Let us consider that passwords are often used repeatedly across multiple platforms. The risk is relatively high that if we can find a password for an email address and log it into a company's email interface, this will lead to significant information leakage that may even result in full compromise. One of the most effective tools that can be used to support linking is [h8mail](https://github.com/khast3x/h8mail).

```shell-session
h8mail -c h8mail_config.ini -t first.lastname@target-company.com
```

Another option is to use the service of [HaveIBeenPwned](https://haveibeenpwned.com/) to determine if any of the email addresses we found are already in databases that contain passwords for them and have been compromised.

### Third Parties

#### **Hosting Provider**

It is always necessary to establish who the `third-party` providers are and what services they provide to the company. This should always be discussed in the meetings before the penetration test.

[SecurityTrails](https://securitytrails.com/) does an excellent job in this research, which we should take advantage of.

If we do not find a hosting provider in the list, it is most likely a `self-hosted system`.

#### Documents

Here we should take a closer look at the providers who could make files available to us. These files can contain valuable information about the company itself and its processes and show us whom they work with to manage the infrastructure. For this search, we use the search engines like Google with the dork `site:` which will limit our searches to the pages we need. The most used providers for documents include, but are not limited to:

| **Provider**        | **Dork**                      |
| ------------------- | ----------------------------- |
| Google Docs         | `site:docs.google.com`        |
| Google Cloud        | `site:cloud.google.com`       |
| Google Storage      | `site:storage.googleapis.com` |
| Microsoft           | `site:docs.microsoft.com`     |
| Amazon Web Services | `site:amazonaws.com`          |

### Technologies in Use

The Home Page

Another beneficial source that can tell us a lot about the target company's website alone is [BuiltWith](https://builtwith.com/). It lists all the technologies that are used by the web server and are deployed on it.

We can get the necessary information about the domain from Shodan using the "domain" parameter and see what details we will find about it. Most of the time, we get a good insight into the systems related to the domain, and sometimes we can even identify the vhosts directly, apart from the subdomains. Sometimes we can even find other domains with different TLDs that extend our attack vector if the scope in the contract allows it.

```
shodan domain target-company.com # repeat for additional interrelated domains bound to a single company
```

Shodan also has databases in which information is stored over a specific and relatively long period. We can use the Shodan CLI to search this database for entries related to our target company.

```shell-session
shodan search target-company | cut -d" " -f1-3
```

## Leaks

Archives, such as [WayBack Machine](https://archive.org/) or [Archive.fo](http://archive.fo/), create so-called `snapshots` or `captures` of websites and store `images`, `videos`, `audio` recordings, `software`, and `files` that have been published on the websites.

A critical information resource is [Pastebin.com](https://pastebin.com/). There, different people share information in text form with each other. Pastebin is a web application that acts as a kind of online clipboard and can store any text, even large text blocks, on the web and make it accessible to third parties via a link.

```
site:pastebin.com [domain/IP/OSINT info] password cisco router
```
