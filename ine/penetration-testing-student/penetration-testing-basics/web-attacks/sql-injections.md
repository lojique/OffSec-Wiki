# SQL Injections

Most web apps use some kind of backend database to store the data they process. To interact with it, Structured Query Language (SQL) is used.

![](<../../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

SQLi attacks allow an unauthorized user to take control over SQL statements used by a web app

Getting control over a backend database means controlling:

* Users' credentials
* Data of the web app
* Credit card numbers
* Shopping transactions
* And much more

## SQL Statements

Example of a SQL statement which queries the database, asking for the `name` and `description` of a record in the `products` table

The selected record will have an `id` value of 9

```sql
SELECT name, description FROM products WHERE id=9;
```

Basic syntax of a **SELECT** statement:

```sql
SELECT <columns list> FROM <table> WHERE <condition>;
```

To learn the basic syntax of a SELECT statement use this resource:

{% embed url="http://www.w3schools.com/sql/sql_intro.asp" %}

It is possible to select constant values:

```sql
SELECT 22, 'string', 0x12, 'another string';
```

The **UNION** command performs a union between two results

```sql
<SELECT statement> UNION <other SELECT statement>;
```

There are two strings you can use to comment a line in SQL:

* \#(the hash symbol)
* \-- (two dashes followed by a space)

```sql
SELECT field FROM table; # this is a comment
select field from table; -- this is another comment
```

### SELECT Example

![](<../../../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

These two queries provide the same result

```sql
select Name, Description from products where ID='1';
select Name, Description from products where Name='Shoes';
```

![](<../../../../.gitbook/assets/image (4) (1) (1).png>)

### UNION Example

This is a UNION example between two SELECT statements:

```sql
SELECT name, Description FROM Products WHERE ID='3' UNION SELECT Username, Password FROM Accounts;
```

![](<../../../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1) (1) (1).png>)

You can also perform a UNION with some chosen data:

```sql
SELECT name, Description FROM Products WHERE ID='3' UNION SELECT 'Example', 'Data';
```

![](<../../../../.gitbook/assets/image (26) (1) (1) (1) (1) (1) (1) (1).png>)

## SQL Queries Inside Web Apps

The previous examples were how to use SQL when querying a database directly from its console

In order to do the same thing from within a web app, the web app must

* **Connect** to the database
* **Submit** the query to the database
* **Retrieve** the results

PHP example code of a connection to a MySQL database and execution of a static query

![](<../../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1).png>)

## Vulnerable Dynamic Queries

Most of the time, queries are not static like in the example above, but rather dynamically built by using users' inputs

Here's an example of a vulnerable dynamic query

![](<../../../../.gitbook/assets/image (24) (1) (1) (1) (1) (1) (1) (1) (1).png>)

As you can see, the code uses user-supplied input to build a query (the id parameter of the GET request) and then submits it to the database

This is very dangerous as a malicious user can exploit the query construction to take control of the database interaction

What if an attacker crafts a <mark style="color:red;">`$id`</mark> value which can change the query to something like: <mark style="color:red;">`' OR 'a' ='a`</mark>

```sql
SELECT Name, Description FROM Products WHERE ID='' OR 'a'='a';
```

This query tells the database to select the items by checking two conditions: The id must be empty or an always true condition

While the first condition is not met, the second condition is an always true condition, thus the database will select all the items in the Products table

So an attacker could exploit the UNION command by supplying the following:

```sql
' UNION SELECT Username, Password FROM Accounts WHERE 'a'='a
```

```sql
SELECT Name, Description FROM Products WHERE ID='' UNION SELECT Username, Password FROM Accounts WHERE 'a'='a';
```

## Finding SQL Injections

Once you find where the injection point is, you can craft a payload to take control over the dynamic query

You must test **every supplied user input** used by the web application

User inputs are can be:

* GET parameters
* POST Parameters
* HTTP Headers
  * User-Agent
  * Cookie
  * Accept
  * ...

Testing an input for SQLi means trying to inject:

* String terminators: **'** and **"**
* SQL commands: **SELECT**, **UNION** and others
* SQL comments: **#** or **--**

### Example

![](<../../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Taking note of the **id** GET parameter, since this is a user input, we can test it to verify if it is vulnerable

We see that id is an injection point

![](<../../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

## Boolean Based SQLi

You want to transform a query in a True/False condition, which reflects its state to the web app output

![always true condition](<../../../../.gitbook/assets/image (21) (1) (1) (1) (1) (1) (1) (1) (1).png>)

![always false condition](<../../../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Since there is no image or view counter, this is clearly an exploitable SQL injection

## UNION Based SQL Injections

Many times, some of the results of a query are directly displayed on the output page

This can be exploited using the **UNION** SQL command

If our payload makes the results of the original query empty, then we can have the results of another, attacker controlled, query shown on the page

```sql
SELECT description FROM item WHERE id='' UNION SELECT user(); -- -';
```

![](<../../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1).png>)

The comment tells MySQL to ignore everything that will be added right after. This is because we don't want the web application to add other strings to our query

The reason for the third dash is because most browsers automatically remove trailing spaces in the URL so, if you need to inject a comment via a GET request, you have to add a character after the trailing space of the comment

### Exploiting UNION SQL Injections

You first need to know how many fields the vulnerable query selects

![](<../../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

We know the injection is here, but we get an error which means the number of fields of the original query and our payload do not match

Trying with two fields seems to work

![](<../../../../.gitbook/assets/image (3) (1) (1).png>)

Trying with three fields verifies if the original query just has two fields

![](<../../../../.gitbook/assets/image (1) (1) (1).png>)

Once we know how many fields are in the query, we can test which fields are part of the output page

We'll inject some known values and checking the results

![](<../../../../.gitbook/assets/image (5) (1) (1) (1) (1).png>)

![](<../../../../.gitbook/assets/image (20) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Now we can exploit the injection. As an example, we'll query for user()

![](<../../../../.gitbook/assets/image (31) (1) (1) (1) (1) (1) (1).png>)

### Avoiding SQL Disaster

Not only can SELECT type queries be vulnerable but also:

```sql
DELETE description FROM items WHERE id=[User Supplied Value]; 
DELETE description FROM items WHERE id='1' or '1'='1';
```

## SQLMap

SQLMap is an open source penetration testing tool that automates the process of detecting and exploiting SQL injection flaws and taking over database servers

Basic syntax

```
sqlmap -u <URL> -p <injection param> [options]
```

To exploit the SQLi in the previous example, the syntax would be:

```
sqlmap -u 'http://victim.site/view.php?id=1141' -p id --technique=U
```

This tests the id parameter of the GET request for view.php and tells SQLMap to use a UNION based SQLi technique

To exploit a POST parameter:

```
sqlmap -u <URL> --data=<POST string> -p <parameter> [options]
```

You can also copy the POST string from a request intercepted with Burp Proxy
