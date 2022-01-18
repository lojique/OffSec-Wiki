# SQL Injections

Most web apps use some kind of backend database to store the data they process. To interact with it, Structured Query Language (SQL) is used.

![](<../../../../.gitbook/assets/image (18).png>)

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

To learn the basic syntax of a SELECT statement use this resource:

{% embed url="http://www.w3schools.com/sql/sql_intro.asp" %}

