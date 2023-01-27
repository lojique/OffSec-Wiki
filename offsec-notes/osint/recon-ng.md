# Recon-ng

## Modules

<pre><code>modules load recon/domains-contacts/whois_pocs
modules load recon/domains-hosts/bing_domain_web
modules load recon/domains-hosts/google_site_web
modules load recon/domains-hosts/brute_hosts
modules load discovery/info_disclosure/interesting_files
modules load recon/domains-hosts/shodan_hostname
modules load recon/netblocks-hosts/shodan_net
modules load recon/domains-hosts/builtwith
modules load recon/companies-multi/whois_miner
<strong>modules load recon/domains-hosts/certificate_transparency
</strong>modules load recon/domains-contacts/hunter_io 
modules load recon/domains-hosts/netcraft
modules load recon/companies-domains/censys_subdomains
modules load recon/companies-multi/censys_org
modules load recon/companies-multi/censys_tls_subjects
modules load recon/contacts-domains/censys_email_to_domains
modules load recon/domains-hosts/hackertarget
modules load recon/companies-multi/github_miner
<strong>modules load recon/profiles-contacts/github_users
</strong><strong>modules load recon/profiles-repositories/github_repos
</strong>modules load recon/repositories-profiles/github_commits
modules load recon/repositories-vulnerabilities/github_dorks
modules load recon/repositories-vulnerabilities/gists_search
modules load recon/profiles-contacts/bing_linkedin_contacts
modules load recon/profiles-contacts/dev_diver
modules load recon/domains-hosts/ssl_san
modules load recon/hosts-hosts/ssltools
modules load recon/ports-hosts/ssl_scan
<strong>modules load recon/domains-vulnerabilities/ghdb
</strong>modules load recon/domains-vulnerabilities/xssed
modules load recon/hosts-ports/binaryedge
modules load recon/hosts-ports/shodan_ip
modules load recon/ports-hosts/migrate_ports
modules load recon/companies-multi/shodan_org
</code></pre>
