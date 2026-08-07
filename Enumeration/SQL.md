
>sqlite3
```sql
sqlite3 db.db
.tables ->  shows all tables
.dump users ->  dumps table users 

---------------
#guid

sqlitebrowser <db>
```


>mysql
```
mysql --host=192.168.163.148  -u root  -p --skip-ssl
mysql -u <user> -p 'password' -h localhost

use <database>;
show tables;
select * from <table>;
```


>mssql

```sql

--mssqlclient built-in helpers 
enum_db
enum_logins
enum_users
enum_impersonate
enum_links

--select db
use <DBName>
-- enum tables in the db
SELECT table_name FROM information_schema.tables;

SELECT * FROM <TABLE_NAME>;
```